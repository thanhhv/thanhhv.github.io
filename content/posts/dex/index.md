+++
title = "Tổng quan về Hợp đồng Thông minh trong DEX và AMM"
date = "2025-05-18T16:00:00+07:00"
draft = false
tags = ["blockchain", "DEX", "smart contract", "Uniswap", "AMM"]
+++

## Hợp đồng thông minh DEX

Trong một sàn giao dịch phi tập trung (DEX) như Uniswap, nhiều hợp đồng thông minh làm việc cùng nhau để cho phép hoán đổi token. Các hợp đồng chính thường bao gồm: **Factory**, **Router**, và **Pair**. Vai trò của từng hợp đồng như sau:

### 🔹 1. Factory Contract
- Là nơi đăng ký chính của các pool thanh khoản (cặp token).
- Tạo ra pool thanh khoản mới khi có thêm cặp giao dịch mới.
- Lưu trữ ánh xạ giữa các cặp token và địa chỉ của hợp đồng Pair tương ứng.

**Các hàm chính:**
- `createPair(tokenA, tokenB)`: Triển khai hợp đồng Pair mới cho cặp tokenA và tokenB.
- `getPair(tokenA, tokenB)`: Trả về địa chỉ hợp đồng Pair.
- `allPairs()`: Trả về danh sách tất cả các cặp đã tạo.

**Ví dụ:**  
Nếu người dùng muốn giao dịch Token A ↔ Token B mà chưa có pool, Factory sẽ tạo hợp đồng Pair mới.

### 🔹 2. Router Contract
- Là hợp đồng chính mà người dùng tương tác để hoán đổi token, thêm/bớt thanh khoản và định tuyến giao dịch.
- Gọi đến Factory để tìm hợp đồng Pair tương ứng.
- Đảm bảo giá tốt nhất bằng cách định tuyến giao dịch qua nhiều cặp token.

**Các hàm chính:**
- `swapExactTokensForTokens(amountIn, minAmountOut, path, recipient, deadline)`: Hoán đổi amountIn của Token A sang Token B.
- `addLiquidity(tokenA, tokenB, amountA, amountB, minA, minB, to, deadline)`: Thêm thanh khoản vào một Pair.
- `removeLiquidity(tokenA, tokenB, liquidity, minA, minB, to, deadline)`: Rút thanh khoản khỏi Pair.

**Ví dụ:**  
Người dùng muốn đổi Token A sang Token B:
- Router kiểm tra tuyến giao dịch tối ưu (trực tiếp hoặc qua nhiều cặp).
- Gọi đến hợp đồng Pair để thực hiện giao dịch.
- Token được chuyển giao tương ứng.

### 🔹 3. Pair Contract (Liquidity Pool)
- Quản lý pool thanh khoản cho một cặp token.
- Sử dụng công thức AMM sản phẩm không đổi:  
  **x * y = k**  
  Trong đó `x` và `y` là lượng token dự trữ, `k` là hằng số không đổi.

**Các hàm chính:**
- `swap(amount0Out, amount1Out, to)`: Thực hiện hoán đổi token.
- `mint(to)`: Mint token LP (liquidity provider).
- `burn(to)`: Burn token LP và trả lại token gốc.
- `getReserves()`: Trả về lượng token dự trữ.

**Ví dụ:**  
Nếu có giao dịch Token A → Token B:
- Người dùng gửi Token A vào hợp đồng Pair.
- Pair tính toán số Token B trả lại dựa vào công thức AMM.
- Gửi Token B cho người dùng, cập nhật dự trữ.

## 🛠 Các hợp đồng khác trong DEX

### 🔹 4. Multicall Contract (tùy chọn)
- Cho phép thực hiện nhiều lệnh gọi hợp đồng trong một giao dịch (ví dụ: kiểm tra nhiều giá cùng lúc).

### 🔹 5. Fee Collection Contract (tùy chọn)
- Nếu DEX thu phí giao thức, sẽ có hợp đồng riêng để thu và phân phối phí.

## 🎯 Cách các hợp đồng phối hợp với nhau

**Tạo cặp token mới:**
- Factory triển khai hợp đồng Pair nếu chưa tồn tại.

**Thêm thanh khoản:**
- Người dùng gọi Router, Router nạp token vào Pair contract.
- Người dùng nhận lại token LP.

**Hoán đổi token:**
- Người dùng gọi Router để hoán đổi.
- Router tìm đúng Pair và thực hiện swap.

**Rút thanh khoản:**
- Người dùng trả lại token LP cho Router, Router rút tài sản từ Pair.

## 🚀 Ví dụ luồng swap

Giả sử bạn muốn hoán đổi 100 USDT → ETH trên DEX:
1. Người dùng gọi:  
   `swapExactTokensForTokens(100 USDT, minETH, [USDT, ETH], recipient, deadline)`
2. Router truy xuất hợp đồng Pair (USDT/ETH) từ Factory.
3. Pair tính toán số ETH trả lại theo công thức AMM.
4. Gửi ETH cho người dùng, USDT thêm vào dự trữ.

## 🔥 Tổng kết

| Hợp đồng | Mục đích |
|----------|---------|
| Factory | Tạo và theo dõi các hợp đồng Pair (pool thanh khoản). |
| Router | Xử lý swap, định tuyến, quản lý thanh khoản. |
| Pair | Giữ token dự trữ, thực hiện swap theo AMM. |
| Multicall (tuỳ chọn) | Gộp nhiều lệnh gọi hợp đồng thành một giao dịch. |
| Fee Collection (tuỳ chọn) | Thu và phân phối phí giao thức. |

## Uniswap V2 - Công thức sản phẩm không đổi và cách tính giá

Uniswap V2 hoạt động theo mô hình **AMM** (Automated Market Maker), nơi tính thanh khoản trong pool được duy trì bởi công thức sản phẩm không đổi:

**x * y = k**

Trong đó:
- `x` = Số lượng Token X trong pool  
- `y` = Số lượng Token Y trong pool  
- `k` = Hằng số không đổi sau mỗi giao dịch  

**Ví dụ:**
- Token X là **ETH**
- Token Y là **USDT**

### Thiết lập ban đầu
Pool có:
- 1000 ETH
- 500,000 USDT

`k = 1000 * 500,000 = 500,000,000`

### Bước 1: Tính giá ban đầu
Giá ETH theo USDT:  
`1 ETH = 500 USDT`

### Bước 2: Giao dịch swap
Người dùng swap 100 ETH → USDT  
ETH trong pool: 1000 → 1100  
USDT giảm để giữ k = 500,000,000

### Bước 3: Tính USDT mới
`1100 * y' = 500,000,000 → y' = 454,545.45 USDT`

### Bước 4: Tính số USDT nhận được
`500,000 - 454,545.45 = 45,454.55 USDT`

### Bước 5: Giá ETH mới
`1 ETH ≈ 412.37 USDT`

## Tóm tắt

| Trạng thái | ETH | USDT | Giá ETH |
|------------|-----|------|---------|
| Trước swap | 1000 | 500,000 | 500 USDT |
| Sau swap   | 1100 | 454,545.45 | 412.37 USDT |

## Kết luận

- **Tác động giá:** Giá ETH giảm vì tỉ lệ thay đổi.
- **Sản phẩm không đổi:** `k` không đổi, đảm bảo thanh khoản.
- **Lợi ích LP:** Kiếm phí nhưng có nguy cơ tổn thất tạm thời.

Bài viết được mình dịch từ **[duyquoc1508](https://github.com/duyquoc1508/documentation?tab=readme-ov-file#dex-smart-contract)**, cám ơn **Quốc** vì đã có 1 bài viết quá chi tiết này.