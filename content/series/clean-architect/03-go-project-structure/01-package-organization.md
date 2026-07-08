+++
title = "Chương 3.1 — Tổ chức Package trong Go: cmd/, internal/, pkg/ và các trường phái"
date = "2026-07-08T02:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 2–3** · Chương này trả lời câu hỏi thực dụng nhất: "project của tôi nên có cấu trúc thư mục thế nào?" — nhưng bằng nguyên lý, không bằng template.

---

## 1. Problem Statement

Go không quy định cấu trúc project (khác Rails/Django). Tự do này tạo ra hai thái cực lỗi:

- **Thiếu cấu trúc**: mọi thứ trong một package `main` 20.000 dòng, hoặc chia file tùy hứng — không có ranh giới nào để compiler bảo vệ.
- **Thừa cấu trúc**: bê nguyên template "golang-standard-project-layout" hay "clean-architecture-template" trên GitHub về cho một service 3 endpoint — 40 thư mục, mỗi feature chạm 9 file, team ghét kiến trúc từ đó.

Cả hai đều xuất phát từ việc coi cấu trúc thư mục là *mục tiêu* thay vì *hệ quả* của các quyết định ranh giới (chương 1.1) và hướng phụ thuộc (chương 2.2).

Nguyên tắc dẫn đường của cả chương: **package là đơn vị API và đơn vị phụ thuộc; thư mục chỉ là nơi package sống.** Thiết kế package trước, thư mục theo sau.

## 2. Ba cơ chế Go cho bạn

### `cmd/` — nhiều binary, một codebase

```
cmd/
├── api/main.go        # HTTP server
├── worker/main.go     # background consumer
└── migrate/main.go    # chạy migration
```

Mỗi thư mục con là một `package main` — một binary. Quy ước (không phải luật ngôn ngữ) nhưng nên theo: nó tách "điểm lắp ráp" (composition root) khỏi "thư viện logic", và làm rõ rằng một hệ thống có thể có nhiều cách chạy cùng một nghiệp vụ — đúng tinh thần "delivery là chi tiết".

`main.go` phải **mỏng**: đọc config, mở kết nối, lắp ráp, chạy, shutdown. Logic trong main không test được và không tái dùng được.

### `internal/` — ranh giới được compiler cưỡng chế

Package dưới `internal/` chỉ import được từ cây thư mục cha của `internal/` đó. Đây là công cụ kiến trúc mạnh nhất của Go:

```
myapp/
├── internal/order/     # chỉ code trong myapp/ import được
└── internal/order/internal/calc/  # chỉ code trong internal/order/ import được (internal lồng nhau!)
```

Dùng cho: toàn bộ code ứng dụng (mặc định!), và `internal/` lồng để bảo vệ chi tiết của từng module khỏi module anh em. Nếu `payment` không được đụng vào ruột của `order`, đặt ruột đó dưới `internal/order/internal/`.

### `pkg/` — dùng thận trọng

`pkg/` mang nghĩa "code cho người ngoài repo import". Với **ứng dụng** (không phải thư viện), khuyến nghị: **không dùng `pkg/`** — mọi thứ vào `internal/`, vì công bố API công khai là cam kết bảo trì mà bạn không cần tự nguyện gánh. Chỉ dùng khi có consumer ngoài repo thật sự. Nhiều codebase dùng `pkg/` làm nơi chứa "utils dùng chung" — đó là anti-pattern `utils` (chương 1.1) đội tên mới.

## 3. Bốn trường phái tổ chức và trade-off

Cùng một hệ thống (order, payment, user), bốn cách bố trí:

### A. Flat — một package

```
internal/app/
├── handlers.go, orders.go, payments.go, users.go, db.go
```

Mọi thứ một package. **Ưu**: zero indirection, refactor nội bộ tự do, tốc độ tối đa. **Nhược**: không có ranh giới nào — mọi hàm gọi được mọi hàm, coupling tự do mọc; không chia ownership được. **Dùng khi**: service < ~3–5 nghìn dòng, 1–2 người, hoặc giai đoạn khám phá domain (ranh giới chưa rõ thì đừng vẽ).

### B. Layer-based — chia theo tầng kỹ thuật

```
internal/
├── handler/   (mọi HTTP handler)
├── service/   (mọi business logic)
├── repository/(mọi truy cập DB)
└── model/     (mọi struct)
```

Phổ biến nhất trong các codebase Go doanh nghiệp — và là trường phái **đáng tránh nhất ở quy mô lớn**. Phân tích theo chương 1.1: cohesion chỉ đạt mức *logical* (gom theo thể loại); mọi feature trải ngang 4 package nên một thay đổi nghiệp vụ = 4 nơi sửa (shotgun surgery); `model/` trở thành điểm coupling toàn cục (mọi package import nó); không thể dùng `internal/` để bảo vệ gì vì mọi tầng cần mọi tầng. Thêm nữa: tên package vô nghĩa (`service.Service`, `handler.Handler`) và dễ import vòng khi hai service gọi nhau.

**Ưu** duy nhất nhưng có thật: quen thuộc, dễ đoán với người mới, đủ tốt cho service nhỏ ít domain logic. **Dùng khi**: CRUD service nhỏ, team quen mô hình MVC.

### C. Feature-based (package by feature) — khuyến nghị mặc định

```
internal/
├── order/
│   ├── order.go          # domain + use case (vòng trong)
│   ├── service.go
│   ├── postgres.go       # adapter — hoặc tách package con khi lớn
│   └── http.go
├── payment/
│   └── ...
└── platform/             # hạ tầng kỹ thuật dùng chung: config, log, dbconn
```

Mỗi package = một năng lực nghiệp vụ. Cohesion đạt mức *functional*; tên package có nghĩa (`order.Service`, `payment.Gateway`); ranh giới nghiệp vụ trùng ranh giới compiler; team ownership rõ (`CODEOWNERS` theo thư mục); và đây là **đường tiến hóa tự nhiên lên modular monolith và microservices** — một feature-package tách thành service riêng dễ hơn nhiều so với gom code từ 4 layer-package.

**Nhược**: phải quyết định "feature nào" — tức phải hiểu domain (nhưng đó là việc đằng nào cũng phải làm); code dùng chung giữa các feature cần chỗ ở và kỷ luật (nếu không sẽ đẻ lại `utils`); quan hệ giữa các feature (order cần user) phải thiết kế chủ động — qua interface hoặc gọi thẳng có kỷ luật.

### D. Feature-based + layer bên trong (Clean Architecture đầy đủ)

```
internal/
├── order/
│   ├── domain/
│   ├── usecase/
│   └── adapter/{httpapi,postgres,kafka}/
└── payment/
    └── ... (tương tự)
```

Chính là cấu trúc chương 2.3, nhân bản theo module. Ranh giới hai trục (chương 1.4): trục nghiệp vụ giữa các module, trục kỹ thuật trong mỗi module. **Dùng khi**: module có domain logic thật, nhiều use case, nhiều delivery/adapter, team ≥ 3–4 người trên codebase. **Nhược**: nghi thức tối đa — với module chỉ có 2 endpoint CRUD, 3 package con là thừa.

### Bảng quyết định

| Tiêu chí | Flat | Layer | Feature | Feature+Layer |
|---|---|---|---|---|
| Service nhỏ, 1–2 dev | ✅ tốt nhất | ⚠️ | ⚠️ thừa nhẹ | ❌ thừa |
| CRUD nhiều bảng, ít rule | ⚠️ | ✅ chấp nhận | ✅ | ⚠️ |
| Domain logic thật, sống lâu | ❌ | ❌ | ✅ | ✅ tốt nhất |
| Nhiều team một codebase | ❌ | ❌ | ✅ | ✅ |
| Tiến hóa lên tách service | ❌ | ❌ khó nhất | ✅ | ✅ |

Điểm mấu chốt: **các mức này là một con đường tiến hóa, không phải bốn lựa chọn ngang hàng.** Bắt đầu Flat → tách Feature khi thấy các cụm thay đổi độc lập → thêm layer trong feature khi module phình. Refactor giữa các mức trong Go rẻ (đổi import path, compiler dẫn đường) — rẻ hơn nhiều so với trả nghi thức thừa suốt hai năm.

## 4. Vertical Slice — biến thể đáng biết

Vertical Slice Architecture (Jimmy Bogard) đẩy feature-based đến cực đoan: đơn vị tổ chức là **use case**, không phải module:

```
internal/order/
├── placeorder/     # tất cả của MỘT use case: handler, logic, SQL
│   └── placeorder.go
├── cancelorder/
│   └── cancelorder.go
└── shared/         # entity Order dùng chung giữa các slice
```

Triết lý: tối đa cohesion theo *trục thay đổi thật* (một yêu cầu thường sửa một use case), chấp nhận duplication giữa các slice thay vì abstraction sớm. Mỗi slice tự chọn mức nghi thức: slice CRUD viết thẳng SQL, slice phức tạp dùng domain model đầy đủ.

**Trade-off với Clean Architecture cổ điển**: Slice thắng về locality of change và cho phép "mỗi use case một kiểu"; thua khi enterprise rules dày (rule chung phải sống ở `shared/` — và `shared/` phình dần thành domain layer, quay về Clean Architecture). Thực tế production: kết hợp — **tổ chức slice cho phần CRUD, domain model chung cho phần lõi nghiệp vụ**. Hai trường phái không loại trừ nhau; chúng khác nhau ở chỗ đặt mặc định.

## 5. Best Practices đặt tên & bố trí

- Tên package: ngắn, số ít, là danh từ nghiệp vụ — `order`, không phải `orders`, `ordermanagement`, `orderservice`.
- Không lặp tên: `order.Service` chứ không `order.OrderService`.
- Cấm các package: `utils`, `common`, `helpers`, `base`, `shared` (trừ khi nội dung thật sự là một khái niệm — khi đó đặt tên theo khái niệm: `money`, `pagination`).
- `platform/` (hoặc `pkg/` nội bộ có kỷ luật) cho hạ tầng kỹ thuật thuần: `platform/postgres` (connection, tx helper), `platform/httpserver`, `platform/kafka`. Phân biệt với domain: platform không biết nghiệp vụ.
- Một package một trách nhiệm *có thể phát biểu trong một câu không có chữ "và"*.
- Kiểm soát quan hệ giữa feature: module A cần module B → gọi qua **API công khai của B** (hàm/interface exported), không với vào `internal/` của B; cân nhắc interface + event khi hai module cần độc lập triển khai (chương 12).

## 6. Anti-patterns

- **Copy "standard layout" mù quáng**: repo 5 file với đủ `api/ assets/ build/ configs/ deployments/ docs/ examples/ scripts/ third_party/ tools/`. Cấu trúc phải mọc theo nhu cầu.
- **`model/` toàn cục**: mọi struct một chỗ → điểm coupling toàn hệ thống, ai đổi gì cũng chạm. Struct sống cùng package sở hữu hành vi của nó.
- **Package theo pattern**: `factory/`, `strategy/`, `observer/` — tổ chức theo *cách viết* thay vì *cái gì*. Không ai tìm business rule trong thư mục `strategy/`.
- **Import vòng giải bằng package thứ ba vô nghĩa**: `order` ↔ `payment` vòng nhau → đẻ `types/` chứa chung — coupling không giảm, chỉ đổi địa chỉ. Giải đúng: xét lại ranh giới (có khi hai module là một), hoặc đảo một chiều bằng interface/event.
- **Một file nghìn dòng vì "một package một file"**: package là đơn vị API; file chia theo chủ đề thoải mái, miễn cùng package.

## 7. Khi nào KHÔNG cần nghĩ về tất cả những điều này

Tool CLI vài trăm dòng, lambda function, service proxy mỏng: một package `main`, vài file, xong. Cấu trúc package là công cụ quản lý *nhiều người, nhiều thay đổi, nhiều năm* — thiếu một trong ba yếu tố đó thì mặc định tối giản luôn đúng.

## Tóm tắt

- Thiết kế package (API + hướng phụ thuộc) trước, thư mục theo sau; `internal/` là hàng rào compiler — dùng tối đa; `pkg/` — hầu như không.
- Mặc định: feature-based; thêm layer bên trong feature khi module đủ dày; flat khi nhỏ; layer-based toàn cục — tránh ở quy mô lớn.
- Cấu trúc là con đường tiến hóa: Flat → Feature → Feature+Layer → (khi cần) tách service.

**Chương tiếp theo:** [Dependency Injection trong Go](/series/clean-architect/04-dependency-injection/01-di-trong-go/)
