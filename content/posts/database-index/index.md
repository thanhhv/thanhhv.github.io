+++
title = "4 Nguyên Tắc Vàng Khi Dùng Index Mà Mình Ước Gì Biết Sớm Hơn"
date = "2026-05-01T20:00:00+07:00"
draft = false
tags = ["database", "postgresql", "index"]
+++

Chào mọi người, trong quá trình làm việc với database, mình thấy rất nhiều trường hợp không hiểu rõ bản chất của index dẫn đến không sử dụng index một cách hiệu quả. Hôm nay, tranh thủ dịp lễ 1/5 không được đi chơi, ở nhà thôi thì ngồi viết một bài blog về vấn đề này vậy. (Lưu ý: Bài viết mình có tham khảo AI sau khi viết để chỉnh chu lại văn phong và ngữ pháp cũng như là verify lại kiến thức của mình nhé, nào gét gô)

> Khi gặp một query chậm, hãy hình dung cách query đó *map* vào index theo 4 nguyên tắc này — đó là cách nhanh nhất để xác định index cần tạo.

---

## Nguyên tắc 1: Nhảy thẳng vào chỗ cần tìm, không lạc đường

Thao tác cơ bản nhất của index: thay vì scan từ đầu đến cuối như đọc cả cuốn sách để tìm một câu, index cho phép database nhảy thẳng đến đúng trang cần đọc.

Ví dụ bảng `orders` của một app đặt đồ ăn, muốn lấy tất cả đơn hàng trong ngày 20/01:

```sql
SELECT * FROM orders WHERE order_date = '2024-01-20';
```

Hình dung index trên `order_date` như một cuốn sách có mục lục:

```
Internal nodes (summary):
[... | 01/01–10/01 | 11/01–20/01 | 21/01–31/01 | ...]
                          │
                          ▼
Leaf nodes: [18/01 | 19/01 | 20/01 | 20/01 | 20/01 | 21/01 | 22/01]
                             ▲
                    Database nhảy thẳng đến đây!
```

Database không scan qua 01/01, 05/01, 10/01... mà nhảy thẳng đến vùng `11/01–20/01` trong internal nodes, rồi tìm chính xác `20/01` trong leaf nodes. Gọn gàng, không vòng vo.

### Hiểu nhầm hay gặp: "Index càng lớn thì query càng chậm"

Không phải vậy đâu! Index có cấu trúc cây (B+ tree), không phải danh sách phẳng. Mỗi tầng internal node chia dữ liệu thành hàng trăm đến hàng nghìn nhánh (branching factor). Kết quả thực tế:

| Số rows       | Tree depth (ước tính) |
| ------------- | --------------------- |
| 1,000         | ~2                    |
| 1,000,000     | ~3                    |
| 1,000,000,000 | ~4                    |

Từ 1 nghìn lên 1 tỷ row mà chỉ thêm 2 bước nhảy — đó là sức mạnh của **O(log n)**. Index đã được tối ưu hoá hàng chục năm cho đúng use case này, đừng lo lắng về kích thước của nó.

---

## Nguyên tắc 2: Một khi đã tìm được chỗ, cứ chạy thẳng một mạch

Sau khi nhảy đến đúng vị trí trong index, database có thể tiếp tục đọc liên tục theo một hướng (ascending hoặc descending) mà không cần nhảy nhót gì thêm. Vì leaf nodes của B+ tree được liên kết với nhau (doubly linked list), di chuyển sang entry tiếp theo cực kỳ rẻ — giống như lật trang sách vậy.

Ví dụ bảng `transactions` của một app ngân hàng, muốn lấy 3 giao dịch có giá trị từ 5 triệu trở lên, sắp xếp tăng dần:

```sql
SELECT * FROM transactions WHERE amount >= 5000000 ORDER BY amount ASC LIMIT 3;
```

```
Index (amount):
[200k | 500k | 1M | 2M | 3.5M | 5M | 6.2M | 8M | 12M | 20M | 50M]
                                     ▲
                    Fast Lookup: nhảy đến amount >= 5M

→ 5M   ✅  (lấy)
→ 6.2M ✅  (lấy)
→ 8M   ✅  (lấy, đủ LIMIT 3 → DỪNG!)
   12M, 20M, 50M... (không cần đọc)
```

Tương tự với hướng ngược — lấy 3 giao dịch lớn nhất dưới 5 triệu:

```sql
SELECT * FROM transactions WHERE amount <= 5000000 ORDER BY amount DESC LIMIT 3;
-- Fast lookup đến 5M, scan ngược: 5M → 3.5M → 2M → DỪNG!
```

**Sức mạnh thực sự khi kết hợp với `LIMIT`:** Không có index, query trên bảng 10 triệu giao dịch phải scan toàn bộ 10M rows → filter → sort → lấy 3. Với index: nhảy đến `5M` → đọc 3 entries → xong. Chênh lệch có thể từ vài giây xuống dưới 1ms — kiểu chênh lệch mà user cảm nhận được luôn.

> **Lưu ý nhỏ:** Scan chỉ đi một hướng trong một lần traversal. Nếu query cần sort theo 2 cột với chiều khác nhau (ví dụ: `ORDER BY amount DESC, created_at ASC`), bạn cần tạo composite index với đúng thứ tự sort đó.

---

## Nguyên tắc 3: Composite Index hoạt động như cái phễu — đổ từ trên xuống, không nhảy cóc

Đây là nguyên tắc quan trọng nhất và cũng dễ hiểu sai nhất. Single-column index thì đơn giản rồi, nhưng **composite index** (multi-column) mới là nơi mang lại cải thiện performance lớn nhất — và cũng là nơi hay bị dùng sai nhất.

### Composite index được sắp xếp như thế nào?

Lấy ví dụ bảng `restaurants` của một app đặt đồ ăn. Index trên `(city, cuisine, rating)` sắp xếp dữ liệu theo thứ tự: sort theo `city` trước, trong mỗi `city` sort theo `cuisine`, trong mỗi `cuisine` sort theo `rating`.

```
Index (city, cuisine, rating):
┌──────────┬───────────────┬────────┐
│ city     │ cuisine       │ rating │
├──────────┼───────────────┼────────┤
│ DN       │ bun-bo        │ 4.1    │
│ DN       │ hai-san       │ 4.5    │
│ DN       │ hai-san       │ 4.7    │
│ HN       │ bun-cha       │ 4.3    │
│ HN       │ pho           │ 4.2    │
│ HN       │ pho           │ 4.8    │
│ HCM      │ banh-mi       │ 4.0    │
│ HCM      │ com-tam       │ 4.6    │ ← target
│ HCM      │ com-tam       │ 4.9    │ ← target
│ HCM      │ lau           │ 4.3    │
└──────────┴───────────────┴────────┘
```

Để ý nhé: trong mỗi `city`, các `cuisine` được sorted. Nhưng nhìn toàn bộ cột `cuisine`, nó **không sorted** globally (bun-bo, hai-san, bun-cha, pho...). Đây chính là lý do tại sao phải đi từ trái sang phải — bỏ qua cột đầu là mất định hướng ngay.

### Phễu hoạt động thế nào?

```sql
-- Tìm quán com-tam ở HCM có rating cao nhất
WHERE city = 'HCM' AND cuisine = 'com-tam' AND rating >= 4.5
```

```
Bước 1: city = 'HCM'         → filter thu hẹp: 4 entries (HCM block)
Bước 2: cuisine = 'com-tam'  → filter thu hẹp: 2 entries
Bước 3: rating >= 4.5        → filter thu hẹp: 1 entry (rating 4.9)
```

Mỗi bước thắt filter lại một lần, đẹp không?

### Query nào dùng được, query nào không?

```sql
-- ✅ Dùng 3/3 bước filter
WHERE city = 'HCM' AND cuisine = 'com-tam' AND rating >= 4.5;

-- ✅ Dùng 2/3 bước filter (bỏ cột cuối — vẫn OK)
WHERE city = 'HCM' AND cuisine = 'com-tam';

-- ✅ Dùng 1/3 bước filter
WHERE city = 'HCM';

-- ❌ KHÔNG dùng được (bỏ qua cột đầu tiên)
WHERE cuisine = 'com-tam';
-- com-tam xuất hiện rải rác ở HCM, HN, DN... không liền nhau trong index

-- ❌ KHÔNG dùng được (bỏ qua cả 2 cột đầu)
WHERE rating >= 4.5;
```

> **Câu thần chú:** *"Từ trái sang phải, không được bỏ qua cột."* — Khắc vào đầu cái này là dùng composite index ngon rồi.

### Hiểu nhầm kinh điển: "Đặt cột selective nhất lên đầu"

Lời khuyên này bay khắp nơi trên internet: *"đặt cột có nhiều distinct values nhất lên đầu index."* Nghe có vẻ hợp lý, nhưng thực ra chưa đủ.

Giả sử bạn đổi thứ tự index thành `(cuisine, rating, city)` vì `cuisine` có nhiều giá trị hơn `city`. Query đầy đủ 3 cột vẫn chạy tốt. Nhưng query `WHERE city = 'HCM'` — kiểu query rất phổ biến khi user mở app và chọn thành phố — **hoàn toàn không dùng được** index này. Tạo index xong mà không dùng được thì tạo làm gì cho nặng máy?

**Cách đúng hơn:** Thứ tự cột phải được quyết định bởi tập hợp các query mà app bạn thực sự chạy, nhằm tối đa hoá số query được phục vụ bởi một index duy nhất.

```sql
-- App đặt đồ ăn thường chạy các query:
-- Q1: WHERE city = 'HCM'                               (rất thường xuyên — user chọn TP)
-- Q2: WHERE city = 'HCM' AND cuisine = 'com-tam'       (thường xuyên — lọc món)
-- Q3: WHERE city = 'HCM' AND cuisine = 'com-tam' AND rating >= 4.5  (ít hơn — lọc chất lượng)

-- Index (city, cuisine, rating)  → phục vụ cả 3 query ✅
-- Index (cuisine, rating, city)  → chỉ Q3 tốt, Q1 không dùng được ❌
-- Index (rating, cuisine, city)  → không phục vụ tốt query nào ❌
```

### Bỏ qua cột giữa: chạy được, nhưng hơi phí

```sql
-- Index: (city, cuisine, rating)
-- Query: WHERE city = 'HCM' AND rating >= 4.5  ← bỏ qua cuisine ở giữa
```

Database vẫn dùng index này, nhưng kém tối ưu:

```
Bước 1: city = 'HCM'    → Fast Lookup ✅
Bước 2: cuisine bị bỏ qua → filter tắc, không thể thu hẹp tiếp
Bước 3: scan TOÀN BỘ entries có city = 'HCM', check từng entry xem rating >= 4.5

[HCM | banh-mi | 4.0] → ❌
[HCM | com-tam | 4.6] → ✅
[HCM | com-tam | 4.9] → ✅
[HCM | lau     | 4.3] → ❌
[HCM | lau     | 4.8] → ✅
→ Đọc 5 entries, giữ 3
```

Index "hoàn hảo" `(city, rating)` chỉ nhảy thẳng vào đúng entries cần thiết. Với bảng lớn — ví dụ HCM có 50,000 quán — scan toàn bộ để filter `rating` là rất lãng phí.

Dù vậy, **skip cột giữa vẫn tốt hơn không có index**: database ít nhất giới hạn được phạm vi scan và filter `rating` ngay trong index mà không cần load row từ table (index-only condition).

### Dọn dẹp index thừa — đừng để database gánh nặng không cần thiết

Mỗi index phải được cập nhật khi có write, nên index thừa = tốn tài nguyên vô ích — đặc biệt đau với những bảng write nhiều như bảng `tickets` của app đặt vé xe.

```
✅ Index (tenant_id, route_id, departure_date) ĐÃ BAO GỒM chức năng của:
   - (tenant_id)
   - (tenant_id, route_id)
   → Xoá 2 index này đi nếu chúng đang tồn tại độc lập

❌ KHÔNG bao gồm:
   - (tenant_id, route_id, seat_class)  → cột cuối khác nhau
   - (route_id, tenant_id)              → thứ tự cột khác nhau
   → Đây là các index độc lập, không thể thay thế nhau
```

Ví dụ thực tế:

```sql
-- Bảng tickets có 4 index:
-- idx_1: (tenant_id)
-- idx_2: (tenant_id, route_id)
-- idx_3: (tenant_id, departure_date)
-- idx_4: (phone_number)

-- Phân tích:
-- idx_1 bị bao gồm bởi idx_2 (same prefix)         → XÓA idx_1
-- idx_2 và idx_3 có prefix giống, cột 2 khác        → GIỮ cả hai
-- idx_4 phục vụ query tìm vé theo SĐT (không cần tenant_id) → GIỮ

-- Kết quả: giữ idx_2, idx_3, idx_4. Xoá idx_1.
```

---

## Nguyên tắc 4: Range Condition — "Cái bẫy" mà ai cũng từng dính ít nhất một lần

Đây là nguyên tắc hay bị bỏ sót nhất nhưng ảnh hưởng performance rất lớn. Khi gặp range condition (`>`, `<`, `>=`, `<=`, `BETWEEN`), database chuyển sang chế độ scan — và từ lúc đó, **filter không thể thu hẹp thêm bằng các cột phía sau**. Tức là mấy cột đứng sau cột range thì... thôi, chỉ filter được thôi, không giúp giảm scan được nữa.

### Tại sao range condition lại "phá vỡ" filter?

Bảng `drivers` của một app gọi xe, muốn tìm tài xế ở HCM có rating cao và đã được xác minh:

```sql
-- Index: (city, rating, is_verified)
WHERE city = 'HCM' AND rating > 4.5 AND is_verified = true
```

```
Index (city, rating, is_verified):
┌──────┬────────┬─────────────┐
│ city │ rating │ is_verified │
├──────┼────────┼─────────────┤
│ HCM  │ 4.1    │ true        │
│ HCM  │ 4.3    │ false       │
│ HCM  │ 4.6    │ false       │ ← rating > 4.5: bắt đầu scan từ đây
│ HCM  │ 4.7    │ true        │ ← is_verified = true  ✅
│ HCM  │ 4.8    │ false       │ ← is_verified = false ❌ (vẫn phải đọc!)
│ HCM  │ 4.8    │ true        │ ← is_verified = true  ✅
│ HCM  │ 4.9    │ false       │ ← is_verified = false ❌ (vẫn phải đọc!)
│ HCM  │ 5.0    │ true        │ ← is_verified = true  ✅
└──────┴────────┴─────────────┘
→ Đọc 6 entries, giữ 3
```

Sau khi `rating > 4.5` bắt đầu scan, các entry `is_verified = false` và `is_verified = true` xen kẽ nhau lộn xộn. Database không thể "nhảy qua" entries không cần thiết — nó phải đọc từng cái và check.

**Đổi thứ tự cột thành `(city, is_verified, rating)` xem sao:**

```
Index (city, is_verified, rating):
┌──────┬─────────────┬────────┐
│ city │ is_verified │ rating │
├──────┼─────────────┼────────┤
│ HCM  │ false       │ 4.3    │
│ HCM  │ false       │ 4.6    │ ← toàn bộ block false → bỏ qua!
│ HCM  │ false       │ 4.8    │
│ HCM  │ false       │ 4.9    │
│ HCM  │ true        │ 4.1    │
│ HCM  │ true        │ 4.7    │ ← Fast Lookup: city='HCM', is_verified=true, rating > 4.5
│ HCM  │ true        │ 4.8    │ ← scan
│ HCM  │ true        │ 5.0    │ ← scan
└──────┴─────────────┴────────┘
→ Đọc đúng 3 entries, không lãng phí
```

Bây giờ: Fast Lookup qua 2 bước filter (`city` → `is_verified`), sau đó mới scan từ `rating > 4.5`. Chỉ đọc đúng những gì cần đọc. Với bảng hàng triệu tài xế thì con số này rõ ràng hơn nhiều:

- **Index sai** `(city, rating, is_verified)`: scan toàn bộ tài xế HCM có `rating > 4.5`, một nửa trong số đó bị lọc ra vì chưa verified → đọc gấp đôi
- **Index đúng** `(city, is_verified, rating)`: chỉ scan tài xế HCM đã verified có `rating > 4.5` → không lãng phí

### Khi có nhiều range condition cùng lúc

```sql
-- Tìm tài xế ở HCM có rating > 4.5 và đã chạy hơn 100 chuyến
WHERE city = 'HCM' AND rating > 4.5 AND total_trips > 100
```

Chỉ một cột range có thể hưởng lợi từ index scan. Cột range thứ hai chỉ dùng để filter thôi:

```sql
-- Index (city, rating, total_trips):
--   city → filter | rating > 4.5 → scan | total_trips > 100 → filter

-- Index (city, total_trips, rating):
--   city → filter | total_trips > 100 → scan | rating > 4.5 → filter

-- Đặt cột nào trước?
-- → Cột nào loại được NHIỀU row hơn thì đặt trước!
-- Ví dụ: 60% tài xế có rating > 4.5, nhưng chỉ 20% có total_trips > 100
-- → Đặt total_trips trước: INDEX (city, total_trips, rating)
```

### Ngoại lệ: Loose Index Scan / Skip Scan

Một số database có khả năng "nhảy qua" entries trong trường hợp đặc biệt (thường với `GROUP BY` + `MIN`/`MAX`):

- **MySQL**: Loose Index Scan
- **Oracle / SQL Server**: Skip Scan
- **PostgreSQL**: Hỗ trợ hạn chế từ PostgreSQL 14+ (incremental sort, partial skip scan trong một số trường hợp với query planner)

Đây là tối ưu hoá đặc biệt, không phải hành vi mặc định. Đừng thiết kế index dựa vào nó — không khéo lại bị hố đấy.

---

## Tóm lại thì nhớ 4 cái này là đủ

| # | Nguyên tắc | Cốt lõi |
|---|-----------|---------|
| 1 | **Nhảy thẳng vào đích** | `=` — fast lookup, không scan lung tung |
| 2 | **Chạy một mạch** | `>=`, `<=`, `BETWEEN` — scan liên tục, kết hợp `LIMIT` cực mạnh |
| 3 | **Phễu trái sang phải** | Composite index — không bỏ qua cột; thứ tự cột phụ thuộc query thực tế |
| 4 | **Range phá filter** | Equality trước, range sau; nhiều range thì đặt cột selective nhất trước |

> **3 điều cần nhớ:**
> 1. Equality trước, Range sau
> 2. Nhiều range condition → cột nào loại được nhiều row hơn thì đứng trước
> 3. Sau cột range đầu tiên, các cột sau chỉ còn tác dụng filter — vẫn hữu ích, nhưng không giới hạn scan range được nữa

---

Bài hơi dài nhưng mình cố gắng viết chi tiết để ai đọc cũng hiểu được, không chỉ dân chuyên database 😄 Nếu bạn thấy chỗ nào mình giải thích chưa rõ, có ví dụ hay hơn, hoặc đơn giản là muốn tranh luận thì cứ thả comment bên dưới nhé — mình đọc hết và reply đàng hoàng chứ không bỏ xó đâu!