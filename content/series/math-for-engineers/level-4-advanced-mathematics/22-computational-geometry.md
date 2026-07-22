+++
title = "Chương 22 — Computational Geometry: Toán của không gian vật lý"
date = "2026-07-20T10:40:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 4 – Advanced Mathematics
> Yêu cầu trước: Chương 07 (Trees), Chương 13 (Divide & Conquer), Chương 17 (Linear Algebra)

---

## 1. Problem Statement

Bạn nhận một ticket tưởng như đơn giản: "API trả về 10 quán cà phê gần người dùng nhất, p99 dưới 50ms." Database có 5 triệu địa điểm, mỗi địa điểm là một cặp (latitude, longitude).

Bản nháp đầu tiên ai cũng viết được: tính khoảng cách từ người dùng đến *từng* địa điểm, sort, lấy 10. Đó là 5 triệu phép tính khoảng cách (mỗi phép có sin/cos nếu tính đúng trên mặt cầu) cộng một lần sort — hàng trăm millisecond cho *một* request. Với 1.000 request/giây, bạn cần một cụm máy chỉ để trả lời câu hỏi "gần đây có gì".

Vấn đề sâu hơn: index B-Tree quen thuộc không cứu được. B-Tree sắp thứ tự theo **một chiều**. Bạn có thể index theo latitude — nhưng hai điểm có latitude gần nhau có thể cách nhau nửa vòng Trái Đất theo longitude. Không gian 2D không có thứ tự toàn phần nào bảo toàn khái niệm "gần nhau". Đây chính là bài toán mà computational geometry sinh ra để giải:

**Làm sao tổ chức và truy vấn dữ liệu không gian sao cho câu hỏi "gần đâu, nằm trong đâu, cắt cái gì" được trả lời mà không phải nhìn toàn bộ dữ liệu?**

Nếu không có công cụ này, mọi tính năng "quanh đây" — tìm tài xế, gợi ý nhà hàng, geofencing, collision detection trong game — đều là O(n) trên mỗi truy vấn, và không scale.

## 2. Trực giác

### Con người tìm quán cà phê như thế nào?

Khi bạn đứng ở quận 1 và muốn tìm cà phê, bạn không duyệt danh bạ toàn quốc rồi loại dần. Bạn làm ngược lại: **thu hẹp không gian trước, nhìn dữ liệu sau**. "Tôi ở quận 1 → chỉ nhìn quận 1 → chỉ nhìn vài phường lân cận." Mọi cấu trúc dữ liệu không gian trong chương này đều là bản mã hóa của trực giác đó: chia không gian thành các vùng lồng nhau, và dùng vị trí truy vấn để **loại bỏ cả vùng** mà không cần nhìn từng điểm bên trong.

So sánh với binary search: binary search loại một nửa *dãy số* mỗi bước nhờ thứ tự tuyến tính. Cấu trúc không gian loại một nửa (hoặc ba phần tư) *mặt phẳng* mỗi bước nhờ phép chia không gian. Cùng một tinh thần chia để trị (chương 13), khác nhau ở đối tượng bị chia.

### Trực giác thứ hai: hình học không cần lượng giác

Phần lớn thuật toán hình học không hề gọi sin, cos hay atan2. Câu hỏi nền tảng của chúng thô sơ hơn nhiều: **đi từ A đến B rồi nhìn về C — C nằm bên trái hay bên phải?** Chỉ với câu hỏi trái/phải này (trả lời bằng vài phép nhân và trừ), ta xây được: kiểm tra hai đoạn thẳng cắt nhau, điểm nằm trong đa giác, convex hull, và hơn thế. Đó là lý do cross product 2D được gọi là "công cụ vạn năng" — và ta sẽ dẫn xuất nó từ determinant của chương 17.

### Nếu không có khái niệm này thì sao?

Không có orientation test, bạn sẽ dùng góc và lượng giác: chậm hơn, và tệ hơn — đầy lỗi làm tròn ở các trường hợp thẳng hàng. Không có spatial index, mọi truy vấn không gian là full scan. Không có distance metric đúng, app của bạn tự tin báo "quán này cách bạn 300m" trong khi thực tế là 3km — lỗi có thật khi ai đó dùng Euclid trên độ kinh/vĩ.

## 3. First Principles

### Khoảng cách là gì? Tùy vào việc bạn di chuyển thế nào

"Khoảng cách" không phải một khái niệm duy nhất — nó là một **họ hàm** thỏa vài tính chất (không âm, đối xứng, bất đẳng thức tam giác). Chọn metric nào là chọn mô hình chuyển động:

| Metric | Công thức | Mô hình | Dùng khi |
|---|---|---|---|
| Euclidean | √((Δx)² + (Δy)²) | bay thẳng trên mặt phẳng | đồ họa, game, vector embedding |
| Manhattan | \|Δx\| + \|Δy\| | đi theo lưới ô vuông | grid pathfinding, chip routing, taxi trong phố bàn cờ |
| Haversine | công thức trên mặt cầu | đi trên bề mặt Trái Đất | mọi thứ dính đến GPS |

**Vì sao Euclidean sai với tọa độ GPS?** Hai lý do độc lập:

1. (lat, lon) là **góc**, không phải khoảng cách. Một độ vĩ ≈ 111km ở mọi nơi, nhưng một độ kinh ≈ 111km × cos(vĩ độ): 111km ở xích đạo, ~55km ở Oslo, 0km ở cực. Cắm thẳng (Δlat, Δlon) vào Pythagoras là cộng táo với cam — sai số tăng theo vĩ độ.
2. Trái Đất cong. Với khoảng cách ngắn (vài km) độ cong bỏ qua được, nhưng đường "thẳng" giữa Hà Nội và San Francisco là một cung tròn lớn, không phải đường thẳng trong hệ (lat, lon).

Haversine giải quyết cả hai bằng cách làm việc trực tiếp với góc trên mặt cầu:

```go
const earthRadiusM = 6371000.0 // bán kính trung bình Trái Đất (mét)

// haversine trả về khoảng cách theo mét giữa hai điểm GPS.
func haversine(lat1, lon1, lat2, lon2 float64) float64 {
    toRad := func(deg float64) float64 { return deg * math.Pi / 180 }
    dLat := toRad(lat2 - lat1)
    dLon := toRad(lon2 - lon1)
    a := math.Sin(dLat/2)*math.Sin(dLat/2) +
        math.Cos(toRad(lat1))*math.Cos(toRad(lat2))*
            math.Sin(dLon/2)*math.Sin(dLon/2)
    return 2 * earthRadiusM * math.Asin(math.Sqrt(a))
}
```

Mẹo production đáng nhớ: khi chỉ cần **so sánh** khoảng cách (tìm điểm gần nhất) chứ không cần giá trị tuyệt đối, bỏ căn bậc hai — so sánh bình phương khoảng cách cho cùng thứ tự và rẻ hơn. Với GPS trong bán kính nhỏ, xấp xỉ equirectangular (nhân Δlon với cos(lat)) đủ chính xác và nhanh gấp nhiều lần Haversine.

### Cross product 2D: một con số trả lời câu hỏi trái/phải

Từ chương 17, determinant của ma trận 2×2 là **diện tích có dấu** của hình bình hành tạo bởi hai vector. Đặt ba điểm A, B, C và hai vector AB, AC:

> cross(A, B, C) = (Bx−Ax)(Cy−Ay) − (By−Ay)(Cx−Ax)

Đây chính là det [AB; AC] — gấp đôi diện tích có dấu của tam giác ABC. Dấu của nó là toàn bộ thông tin định hướng:

```
cross > 0: C bên TRÁI đường AB     cross < 0: C bên PHẢI     cross = 0: thẳng hàng

        C                                A───────B
       ╱                                      ╲
      ╱   (rẽ trái — ngược kim đồng hồ)        ╲  C  (rẽ phải — thuận kim đồng hồ)
 A───────B
```

```go
type Point struct{ X, Y float64 }

// cross trả về tích có hướng của (b-a) và (c-a):
// > 0 nếu a→b→c rẽ trái, < 0 nếu rẽ phải, = 0 nếu thẳng hàng.
func cross(a, b, c Point) float64 {
    return (b.X-a.X)*(c.Y-a.Y) - (b.Y-a.Y)*(c.X-a.X)
}
```

Ba phép trừ, hai phép nhân, một phép trừ — không lượng giác, không chia, không căn. Với tọa độ nguyên, kết quả **chính xác tuyệt đối** (chỉ cần cẩn thận overflow). Đây là viên gạch của mọi thứ phía sau: convex hull hỏi "có rẽ phải không?", segment intersection hỏi "hai đầu đoạn này nằm hai phía không?", diện tích đa giác là tổng các cross. Nếu bạn thấy mình gọi `atan2` trong một thuật toán hình học, hãy dừng lại — 90% khả năng cross product làm được việc đó nhanh hơn và chính xác hơn.

## 4. Mathematical Model

### Hai đoạn thẳng cắt nhau

Đoạn AB và đoạn CD cắt nhau khi và chỉ khi **C, D nằm hai phía của đường AB, và A, B nằm hai phía của đường CD**. "Hai phía" dịch thẳng ra ngôn ngữ cross product:

```go
// segmentsIntersect kiểm tra hai đoạn AB và CD có cắt nhau không
// (bỏ qua các trường hợp suy biến thẳng hàng để giữ code gọn).
func segmentsIntersect(a, b, c, d Point) bool {
    d1 := cross(a, b, c) // C ở phía nào của AB?
    d2 := cross(a, b, d) // D ở phía nào của AB?
    d3 := cross(c, d, a) // A ở phía nào của CD?
    d4 := cross(c, d, b) // B ở phía nào của CD?
    return d1*d2 < 0 && d3*d4 < 0
}
```

Điều kiện `d1*d2 < 0` nghĩa là hai dấu trái nhau — đúng bản chất "hai phía". Trường hợp bằng 0 (đầu mút chạm nhau, đoạn thẳng hàng chồng lên nhau) cần xử lý riêng bằng kiểm tra bounding box; trong phỏng vấn hãy *nói ra* rằng bạn biết các trường hợp này tồn tại.

### Point in polygon: ray casting và định lý Jordan

Điểm P có nằm trong đa giác không? Bắn một tia từ P sang phải đến vô cực và **đếm số lần tia cắt cạnh đa giác**: lẻ → trong, chẵn → ngoài.

```
            ┌────────────┐
            │            │
      P ●───┼────────────┼───────→  (2 giao điểm: chẵn → P ngoài? Không —
            │            │           P ở ngoài hình, đúng là chẵn)
            │   Q ●──────┼───────→  (1 giao điểm: lẻ → Q trong)
            └────────────┘
```

Vì sao đúng? Định lý đường cong Jordan nói: một đường cong khép kín không tự cắt chia mặt phẳng thành đúng hai miền — trong và ngoài. Mỗi lần tia xuyên qua biên là một lần đổi miền. Xuất phát từ P và kết thúc ở vô cực (chắc chắn "ngoài"), số lần đổi miền lẻ nghĩa là P khởi đầu ở "trong". Không cần chứng minh Jordan (chứng minh chặt khó bất ngờ) — trực giác "mỗi lần xuyên tường là một lần đổi trong/ngoài" là đủ để tin và để cài đặt.

```go
// pointInPolygon dùng ray casting: đếm số cạnh cắt tia ngang từ p sang phải.
func pointInPolygon(p Point, poly []Point) bool {
    inside := false
    n := len(poly)
    for i, j := 0, n-1; i < n; j, i = i, i+1 {
        a, b := poly[i], poly[j]
        // Cạnh (a,b) có vắt qua đường ngang y = p.Y không?
        if (a.Y > p.Y) != (b.Y > p.Y) {
            // Hoành độ giao điểm của cạnh với đường ngang đó
            xCross := a.X + (p.Y-a.Y)*(b.X-a.X)/(b.Y-a.Y)
            if p.X < xCross {
                inside = !inside
            }
        }
    }
    return inside
}
```

Điều kiện `(a.Y > p.Y) != (b.Y > p.Y)` dùng so sánh chặt/không chặt lệch nhau một cách cố ý — đó là mẹo chuẩn để đỉnh nằm đúng trên tia không bị đếm hai lần. O(n) mỗi truy vấn; geofencing production sẽ lọc trước bằng bounding box hoặc spatial index rồi mới chạy ray casting trên số ít đa giác ứng viên.

### Số thực và tội lỗi của dấu bằng

Hình học tính toán là nơi lỗi float lộ mặt nhiều nhất, vì các quyết định *rời rạc* (trái/phải/thẳng hàng) được rút ra từ phép tính *liên tục*. Quy tắc sống còn: không bao giờ so sánh float bằng `==`; dùng epsilon cho dữ liệu thực (`math.Abs(v) < 1e-9`), hoặc — tốt hơn khi có thể — dùng **tọa độ nguyên** để cross product chính xác tuyệt đối. Các thư viện nghiêm túc (CGAL) dùng exact arithmetic có chọn lọc; ở tầng ứng dụng, chọn epsilon theo thang đo dữ liệu (1e-9 độ GPS ≈ 0.1mm — quá nhỏ; 1e-6 hợp lý hơn).

## 5. Thuật toán

### Convex Hull: Andrew's monotone chain

Convex hull là "sợi dây chun bọc quanh tập điểm" — đa giác lồi nhỏ nhất chứa mọi điểm. Andrew's monotone chain xây nó trong O(n log n), toàn bộ chi phí nằm ở bước sort:

1. Sort điểm theo (x, y).
2. Quét trái→phải xây **nửa dưới**: thêm từng điểm, chừng nào ba điểm cuối rẽ *trái* (cross ≥ 0) thì pop điểm giữa — vì điểm đó lõm vào trong.
3. Quét phải→trái xây **nửa trên** tương tự. Ghép hai nửa.

```go
// convexHull trả về bao lồi theo chiều ngược kim đồng hồ (monotone chain).
func convexHull(pts []Point) []Point {
    if len(pts) < 3 {
        return pts
    }
    sort.Slice(pts, func(i, j int) bool {
        if pts[i].X != pts[j].X {
            return pts[i].X < pts[j].X
        }
        return pts[i].Y < pts[j].Y
    })

    build := func(order []Point) []Point {
        var h []Point
        for _, p := range order {
            // Invariant: mọi bộ ba liên tiếp trong h đều rẽ phải (lồi).
            // Pop điểm giữa chừng nào việc thêm p phá vỡ invariant đó.
            for len(h) >= 2 && cross(h[len(h)-2], h[len(h)-1], p) <= 0 {
                h = h[:len(h)-1]
            }
            h = append(h, p)
        }
        return h
    }

    lower := build(pts)
    reversed := make([]Point, len(pts))
    for i, p := range pts {
        reversed[len(pts)-1-i] = p
    }
    upper := build(reversed)
    // Bỏ điểm cuối mỗi nửa (trùng điểm đầu nửa kia)
    return append(lower[:len(lower)-1], upper[:len(upper)-1]...)
}
```

Invariant làm nên tính đúng: **mọi bộ ba điểm liên tiếp trên chuỗi đang xây luôn rẽ cùng một hướng** — vòng `for` bên trong khôi phục invariant mỗi khi thêm điểm phá vỡ nó, hệt như cách ta chứng minh vòng lặp đúng ở chương 03. Mỗi điểm được push một lần và pop tối đa một lần, nên phần quét là O(n) amortized; tổng O(n log n) do sort. Đổi `<= 0` thành `< 0` nếu muốn giữ các điểm thẳng hàng trên biên.

Ứng dụng: bounding volume cho collision detection (kiểm tra hull trước, hình chi tiết sau), phát hiện outlier (điểm trên hull là ứng viên cực trị của mọi hướng chiếu), tính đường kính tập điểm (rotating calipers trên hull), vẽ vùng phủ sóng/vùng hoạt động của đội xe.

### KD-Tree: binary search cho không gian

KD-Tree là BST (chương 07) mà mỗi tầng so sánh theo một trục **xen kẽ**: tầng 0 chia theo x, tầng 1 theo y, tầng 2 lại x... Mỗi node là một điểm kiêm một đường cắt chia không gian làm đôi:

```
         chia theo x                      (5,4)          ← tầng 0: trục x
      │                                  ╱     ╲
   A  │   B          →           (2,6)         (8,1)     ← tầng 1: trục y
      │                          ╱   ╲          ...
   ───┼─── chia theo y       (1,3)  (4,7)                ← tầng 2: trục x
      │
```

Nearest neighbor search là phần đáng giá nhất — nó minh họa **pruning** (cắt tỉa) thuần khiết:

1. Đi xuống như tìm kiếm BST, đến lá — có ứng viên đầu tiên, bán kính r = khoảng cách tốt nhất hiện tại.
2. Quay ngược lên (backtrack). Ở mỗi node, hỏi: "quả cầu bán kính r quanh điểm truy vấn có **vắt qua mặt phẳng cắt** không?" Tức là |tọa độ truy vấn − tọa độ cắt| < r?
3. Không vắt qua → **toàn bộ nửa kia của không gian không thể chứa điểm tốt hơn** — bỏ cả nhánh. Vắt qua → phải thăm nhánh kia.

Đây chính là trực giác "loại cả quận thay vì từng quán" được hình thức hóa. Average case O(log n) với điểm phân bố đều — vì bán kính r co lại nhanh và hầu hết nhánh bị cắt.

**Nhưng KD-Tree suy biến ở chiều cao — curse of dimensionality.** Trong không gian d chiều, thể tích tập trung ở "vỏ": khoảng cách giữa điểm gần nhất và xa nhất co lại về nhau khi d tăng, nên bán kính r hầu như luôn vắt qua mọi mặt cắt → pruning thất bại → thuật toán thăm gần hết cây, tức O(n) với hằng số tệ hơn cả quét thẳng. Quy tắc thực dụng: KD-Tree tốt đến d ≈ 10–20; với embedding 768 chiều của ML, người ta bỏ tìm kiếm chính xác và dùng **approximate** nearest neighbor (HNSW, IVF — họ hàng tinh thần của chương 23: đổi độ chính xác lấy tốc độ).

### Quadtree và Geohash: chia tư và làm phẳng

**Quadtree** chia hình vuông thành 4 phần tư, đệ quy ở ô nào chứa nhiều hơn k điểm — mật độ dữ liệu quyết định độ sâu, vùng trống không tốn gì. Đây là cấu trúc quen thuộc của game engine và bản đồ tile.

**Geohash** đi một nước cờ khác thường: biến bài toán 2D về 1D để **tái sử dụng index thường**. Chia đôi dải kinh độ: nửa trái bit 0, nửa phải bit 1. Làm tương tự với vĩ độ. Đan xen bit kinh/vĩ, mã hóa base32 → chuỗi như `w3gvk1t`. Tính chất vàng: **hai geohash chung prefix càng dài thì càng gần nhau** — nên "tìm quanh đây" trở thành query prefix trên B-Tree hay Redis sorted set, những công cụ đã có sẵn.

Nhưng chiều ngược lại **không đúng**: gần nhau không suy ra chung prefix. Hai điểm sát nhau nhưng nằm hai bên một ranh giới chia (kinh tuyến gốc là ví dụ cực đoan) có geohash khác nhau ngay từ ký tự đầu. Fix chuẩn: luôn query **ô chứa điểm cộng 8 ô lân cận**. Quên edge case này là bug kinh điển của mọi hệ thống geo tự chế — "quán bên kia đường không hiện ra".

**R-Tree** (mức giới thiệu): thay vì chia không gian cố định, R-Tree gom các **bounding rectangle của chính dữ liệu** thành cây — các hình chữ nhật được phép chồng lấn. Vì lưu rectangle chứ không chỉ điểm, nó xử lý được đối tượng có kích thước (đường, vùng, polygon) — lý do nó là xương sống của index GiST trong PostGIS. **Uber H3** (mức nhắc): lưới lục giác toàn cầu — lục giác có khoảng cách đến 6 láng giềng đều nhau, không có "láng giềng chéo" mập mờ như ô vuông, hợp cho phân tích mật độ và pricing theo vùng.

| Cấu trúc | Ý tưởng chia | Mạnh | Yếu |
|---|---|---|---|
| KD-Tree | mặt phẳng cắt xen kẽ trục | k-NN chính xác, in-memory | chiều cao, dữ liệu động |
| Quadtree | chia tư đệ quy theo mật độ | game, tile map, cập nhật cục bộ | không cân bằng khi dữ liệu lệch |
| Geohash | 2D → 1D string, prefix = gần | dùng lại index thường, dễ vận hành | edge case ranh giới, ô méo theo vĩ độ |
| R-Tree | gom bounding box của dữ liệu | đối tượng có kích thước, disk-based | chồng lấn làm chậm khi dữ liệu tệ |

## 6. Trade-off

**Chính xác vs nhanh.** Haversine đúng đến mô hình cầu (~0.5% sai số so với ellipsoid); equirectangular sai hơn chút nhưng đủ cho "quán trong bán kính 5km" và rẻ hơn nhiều; so sánh bình phương khoảng cách miễn phí nhất. Chọn theo yêu cầu: tính tiền cước taxi cần đúng, ranking kết quả chỉ cần đúng *thứ tự*.

**Cấu trúc cây vs mã hóa 1D.** KD-Tree/R-Tree cho truy vấn chính xác và k-NN thật sự, nhưng là code phức tạp phải tự vận hành (hoặc dựa vào database hỗ trợ). Geohash "kém thông minh" hơn — nhưng chạy trên Redis/B-Tree có sẵn, dễ shard, dễ debug bằng mắt. Trong hệ phân tán, cấu trúc *ngu mà phẳng* thường thắng cấu trúc *khôn mà cây*: cây khó cắt ngang qua nhiều máy, prefix string thì shard tự nhiên.

**Build một lần vs cập nhật liên tục.** KD-Tree cân bằng xây O(n log n) nhưng insert/delete làm nó lệch dần — hợp với dữ liệu tĩnh (POI, bản đồ), rebuild định kỳ. Tài xế di chuyển 4 giây/lần cập nhật thì hoặc dùng geohash (update = đổi key), hoặc quadtree với ô lỏng, chứ đừng KD-Tree.

**Khi nào KHÔNG nên dùng gì cả?** Dưới ~10.000 điểm, quét tuyến tính với bình phương khoảng cách chạy trong vài chục microsecond — cache locality của một slice phẳng đánh bại mọi cấu trúc cây (bài học hằng số của chương 10). Đừng dựng PostGIS cho bảng 2.000 cửa hàng; một lần full scan mỗi request là hoàn toàn ổn.

## 7. Production Applications

**PostGIS / PostgreSQL** — `CREATE INDEX ON places USING GIST(geom)` dựng một R-Tree (trên nền GiST). Query `ORDER BY geom <-> :point LIMIT 10` chạy k-NN **ngay trong khi duyệt index**: toán nằm ở phép ước lượng "khoảng cách nhỏ nhất khả dĩ từ điểm truy vấn đến bounding box" — chính là điều kiện pruning của mục 5, thực thi trên trang disk thay vì node bộ nhớ. `ST_Contains` cho geofencing là point-in-polygon với bước lọc bounding box trước.

**Elasticsearch geo query** — `geo_distance` filter dùng BKD-tree (họ hàng KD-Tree tối ưu cho disk, ghi theo block) từ Lucene; trước đó nhiều năm là geohash prefix trên inverted index. Cùng một sản phẩm đã đi qua cả hai nhánh trade-off của mục 6.

**Uber / Grab matching tài xế** — bài toán "tài xế gần nhất" ở tần suất cập nhật cực cao. Uber công khai dùng H3: mỗi tài xế thuộc một cell, tìm quanh điểm đón = duyệt cell chứa nó + các vòng cell lân cận (k-ring). Toán nằm ở thiết kế lưới: lục giác cho khoảng cách láng giềng đồng đều, hierarchy cell cho phép zoom mức phân giải. Surge pricing tính trên cùng lưới đó.

**Game collision detection** — mỗi frame 16ms phải trả lời "cặp vật thể nào chạm nhau" giữa hàng nghìn vật thể; O(n²) cặp là bất khả thi. Broad phase: quadtree (hoặc spatial hash grid) gom vật thể theo vùng, chỉ các vật thể chung ô mới thành cặp ứng viên. Narrow phase: AABB overlap test — hai hình chữ nhật giao nhau khi và chỉ khi chúng chồng lấn trên *cả hai trục*, bốn phép so sánh. Cross product xuất hiện tiếp ở SAT (separating axis theorem) cho hình lồi xoay.

**Bản đồ số & logistics** — snap GPS nhiễu vào đoạn đường gần nhất (map matching): chiếu điểm lên đoạn thẳng là bài toán tích vô hướng (chương 17); tìm đoạn ứng viên là truy vấn R-Tree trên các đoạn đường.

## 8. Interview

**K Closest Points to Origin (LeetCode 973)** — bài geo phổ biến nhất. Ba lời giải tăng dần: sort toàn bộ O(n log n); **max-heap kích thước k** O(n log k) — giữ k điểm tốt nhất, điểm mới đánh bật đỉnh heap nếu gần hơn, hợp khi dữ liệu là stream; **quickselect** O(n) average — partition quanh pivot theo bình phương khoảng cách (chương 16). Điểm cộng lớn trong phỏng vấn: nói rõ *không cần* `math.Sqrt` vì so sánh bình phương bảo toàn thứ tự — người phỏng vấn nghe câu này biết ngay bạn từng làm việc thật với hình học.

**Max Points on a Line (LeetCode 149)** — với mỗi điểm gốc, gom các điểm khác theo slope. Bẫy float: slope 1/3 không biểu diễn chính xác được, hai slope "bằng nhau" có thể khác nhau ở bit cuối. Lời giải sạch: dùng cặp nguyên (dy/g, dx/g) với g = gcd (chương 19) làm key thay vì float — hoặc kiểm tra thẳng hàng bằng cross product = 0 với số nguyên, chính xác tuyệt đối.

**Erect the Fence (LeetCode 587)** — convex hull nguyên bản, yêu cầu giữ điểm thẳng hàng trên biên (đổi dấu so sánh cross như ghi chú ở mục 5). Ít gặp nhưng là bài kiểm tra "bạn có cài được monotone chain không hay chỉ thuộc tên".

**Valid Square (LeetCode 593)** — tính 6 bình phương khoảng cách giữa 4 điểm; hình vuông ⟺ 4 giá trị nhỏ bằng nhau (cạnh), 2 giá trị lớn bằng nhau (chéo), và cạnh ≠ 0. Lỗi phổ biến: quên case 4 điểm trùng nhau, hoặc dùng khoảng cách có căn rồi so sánh float bằng `==`.

**Lỗi tư duy hay gặp nhất:**

- So sánh float bằng `==` — trong hình học, mọi so sánh bằng phải là epsilon hoặc số nguyên.
- Dùng Euclidean trên (lat, lon) thô — sai lệch theo cos(vĩ độ), bug "gần mà không thấy" ở vĩ độ cao.
- Dùng `atan2` để sort theo góc hay so slope khi cross product làm được cùng việc không lỗi làm tròn.
- Quên trường hợp suy biến: điểm trùng, ba điểm thẳng hàng, đa giác có đỉnh nằm đúng trên tia ray casting.

**Từ phỏng vấn sang production:** K Closest Points chính là "tìm 10 quán cà phê" của mục 1 — thu nhỏ. Khác biệt duy nhất: production không được phép nhìn cả n điểm, nên heap/quickselect được thay bằng spatial index. Nhận ra ánh xạ đó là biết mình đang thiếu công cụ gì.

## 9. Anti-pattern

**Tự chế công thức khoảng cách GPS.** `sqrt(dlat² + dlon²) * 111000` xuất hiện trong nhiều codebase thật — chạy "có vẻ đúng" ở demo gần xích đạo, sai 30–50% khi mở thị trường ở vĩ độ cao. Dùng Haversine hoặc thư viện; nếu xấp xỉ, phải là xấp xỉ *có chủ đích* với sai số đã tính.

**Geohash không query ô lân cận.** Chỉ query prefix của ô chứa user → mất mọi kết quả bên kia ranh giới ô. Triệu chứng: kết quả "nhấp nháy" khi user di chuyển vài mét qua biên geohash. Luôn 8 láng giềng + ô trung tâm.

**KD-Tree cho embedding trăm chiều.** "k-NN thì dùng KD-Tree" là phản xạ đúng ở 2D và sai hoàn toàn ở 512D — pruning chết vì curse of dimensionality, chậm hơn brute force SIMD. Chiều cao → ANN (HNSW/IVF) hoặc chấp nhận quét có vector hóa.

**Epsilon một-cỡ-cho-tất-cả.** `1e-9` cho tọa độ mét là hợp lý, cho tọa độ độ GPS là quá chặt, cho tọa độ pixel game đã scale là quá lỏng. Epsilon phải chọn theo thang đo và sai số tích lũy của pipeline — nếu không, "thẳng hàng" thành ngẫu nhiên.

**Dựng spatial database cho bài toán 5.000 điểm.** Chi phí vận hành PostGIS/ES cluster để thay một vòng for chạy 50µs. Anti-pattern kinh điển của chương 10 mặc áo hình học: tối ưu bậc tăng trưởng khi n bé và không bao giờ lớn.

## 10. Best Practices

**Nên:**

- Chọn metric trước, cấu trúc sau: GPS → Haversine (hoặc equirectangular có chủ đích); so sánh thứ hạng → bình phương khoảng cách, không căn.
- Ưu tiên cross product cho mọi câu hỏi định hướng/thẳng hàng/diện tích; giữ tọa độ nguyên khi có thể để được số học chính xác.
- Với "tìm quanh đây" production: dùng index có sẵn của hệ đang vận hành (PostGIS GiST, ES geo, Redis GEO) trước khi nghĩ đến tự cài — spatial index tự chế là nguồn bug ranh giới vô tận.
- Lọc thô rồi mới tính tinh: bounding box / geohash cell trước, Haversine + point-in-polygon sau — kiến trúc hai pha broad/narrow của game engine đúng cho cả backend.
- Viết test cho trường hợp suy biến ngay từ đầu: điểm trùng, thẳng hàng, điểm trên biên, qua kinh tuyến 180°.

**Không nên:**

- Không so sánh float bằng `==`; không dùng float làm key map để gom theo slope/góc.
- Không dùng KD-Tree cho dữ liệu cập nhật liên tục hoặc chiều cao.
- Không tin kết quả demo ở một vĩ độ — dữ liệu geo phải test ở nhiều vùng địa lý.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Vì sao pruning của KD-Tree hiệu quả ở 2D nhưng vô dụng ở 500 chiều — cụ thể điều kiện cắt nhánh thất bại ở đâu?
2. Geohash chung prefix suy ra gần nhau, nhưng gần nhau không suy ra chung prefix — hệ quả kỹ thuật của bất đối xứng này là gì và fix chuẩn ra sao?
3. Trong monotone chain, invariant nào được duy trì trên stack, và vì sao mỗi điểm chỉ bị pop tối đa một lần lại cho O(n) sau khi sort?

---

*Chương tiếp theo: [23 — Probabilistic Data Structures](/series/math-for-engineers/level-5-production/23-probabilistic-data-structures/), nơi ta đẩy trade-off "đổi chính xác lấy tài nguyên" đến cực hạn: đếm tỷ phần tử bằng 12KB và trả lời "có thể có / chắc chắn không".*
