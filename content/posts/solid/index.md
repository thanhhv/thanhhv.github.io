+++
title = "Bộ nguyên tắc SOLID trong lập trình là gì? (Phiên bản kiếp hiệp)"
date = "2025-09-10T15:00:00+07:00"
draft = false
tags = ["solid"]
+++

Hôm nay, trong một buổi chiều cuối thu ở Sài Gòn nhộn nhịp, có một chàng trai đang ngồi ôn lại kiến thức để chuẩn bị tham gia phỏng vấn trong căn nhà nhỏ. Bên ngoài cửa sổ, trời đang đổ những cơn mưa rào kèm những đợt sấm chớp lan man. Lòng ta lại suy tư về một thứ gì đó muốn được viết ra cho nhẹ nỗi lòng, thôi thì ta hãy cùng viết về **SOLID** nhé.

![soli](solid.png)

---

## 1. Truyền thuyết về SOLID trong giang hồ

Trong giang hồ lập trình, có một bộ tâm pháp được gọi là **SOLID**, truyền rằng ai lĩnh hội được thì code ắt tinh thông, hệ thống vững chãi, mở rộng vô biên. Bộ tâm pháp này gồm năm chiêu thức, mỗi chiêu đều ẩn chứa huyền cơ:

### 1. Nhất Dụng Quy Tắc (Single Responsibility Principle)

Mỗi cao thủ chỉ nên luyện một môn võ công, không nên tạp luyện đủ đường. Một class chỉ nên có một trách nhiệm, như kiếm khách chỉ dùng kiếm, không thể vừa múa kiếm vừa gõ trống. Giống như Kiều Phong chỉ cần một bộ Giáng Long Thập Bát Chưởng cũng đủ để xưng bá võ lâm khiến người người nể phục.


![kieuphong](kieuphong.png)

### 2. Khai Bế Quy Tắc (Open/Closed Principle)

Tâm pháp cần mở rộng để thêm chiêu thức mới, nhưng không được tùy tiện thay đổi căn cơ gốc rễ. Code cho phép mở rộng nhưng hạn chế sửa đổi, như môn phái có thể thu nhận thêm đệ tử nhưng tôn chỉ tổ sư thì bất biến.

### 3. Thế Thân Quy Tắc (Liskov Substitution Principle)

Đệ tử xuất sư, thay thế được sư phụ, hành tẩu giang hồ không làm loạn môn pháp. Class con thay thế class cha mà không phá hỏng chương trình, như người nối nghiệp vẫn giữ trọn đạo nghĩa tông môn.

### 4. Giao Diện Phân Ly Quy Tắc (Interface Segregation Principle)

Bí kíp không nên viết dài dòng, bắt đệ tử học cả ngàn chiêu không cần thiết. Interface nên gọn nhẹ, tách nhỏ, ai luyện cái gì thì chỉ học đúng cái đó, tránh việc phải học cả những chiêu không dùng đến.

### 5. Nghịch Hướng Quy Tắc (Dependency Inversion Principle)

Cao thủ chân chính không phụ thuộc vào vũ khí tầm thường, mà nắm được đạo lý võ học. Module cấp cao không phụ thuộc trực tiếp vào cấp thấp, cả hai đều dựa trên trừu tượng. Như tướng quân cầm quân không dựa vào loại giáo mác cụ thể, mà dựa vào binh pháp.

---

Thế là năm chiêu tâm pháp **SOLID** đã bày ra. Người trong võ lâm lập trình nếu ngộ được thì đường code sẽ như giang sơn vững chắc, dễ sửa đổi, dễ mở rộng, không sợ bug tà đạo quấy nhiễu.

Và cho đến ngày nay, trải qua hàng trăm năm tuổi, bộ tâm pháp ấy vẫn còn lưu giữ được những giá trị bền vững. Dẫu thời thế đã đổi thay, cục diện Nam Đế Bắc Cái, Đông Tà Tây Độc đã khép lại, thì thời đại của những anh hùng bàn phím (lập trình viên) lại xuất hiện. Và ta sẽ tiếp tục nói về cách ứng dụng của SOLID trong lập trình.

---

## 2. Chi tiết về SOLID

**SOLID** là bộ 5 nguyên tắc thiết kế hướng đối tượng giúp code dễ bảo trì, mở rộng và giảm phụ thuộc lẫn nhau.

### 2.1 Single Responsibility Principle (SRP)

- **Ý nghĩa:** Mỗi class/module chỉ nên có một lý do để thay đổi.
- **Hiểu đơn giản:** Một class chỉ làm một việc duy nhất.
- **Ví dụ**: `InvoicePrinter` chỉ in hóa đơn, còn tính toán hóa đơn thì để `InvoiceCalculator`.

```ts
// ❌ Sai: Class làm quá nhiều việc
class Invoice {
  calculateTotal() {
    /* logic tính toán */
  }
  printInvoice() {
    /* logic in ấn */
  }
  saveToDB() {
    /* logic lưu DB */
  }
}

// ✅ Đúng: Tách ra từng trách nhiệm riêng
class InvoiceCalculator {
  calculateTotal() {
    /* logic tính toán */
  }
}

class InvoicePrinter {
  print(invoice: InvoiceCalculator) {
    /* logic in ấn */
  }
}

class InvoiceRepository {
  save(invoice: InvoiceCalculator) {
    /* logic lưu DB */
  }
}
```

---

### 2.2 Open/Closed Principle (OCP)

- **Ý nghĩa:** Code nên mở rộng được nhưng hạn chế chỉnh sửa trực tiếp.
- **Hiểu đơn giản:** Khi có yêu cầu mới → thêm class/method thay vì sửa code cũ.
- **Ví dụ:** Thêm class mới để mở rộng tính năng thay vì chỉnh sửa class cũ.

```ts
// ❌ Sai: Phải sửa code khi thêm shape mới
class AreaCalculator {
  calculate(shape: any) {
    if (shape.type === "circle") return Math.PI * shape.radius ** 2;
    if (shape.type === "square") return shape.side ** 2;
    // thêm loại shape khác -> phải sửa class này
  }
}

// ✅ Đúng: Mở rộng bằng cách thêm class, không sửa code cũ
interface Shape {
  area(): number;
}

class Circle implements Shape {
  constructor(public radius: number) {}
  area() {
    return Math.PI * this.radius ** 2;
  }
}

class Square implements Shape {
  constructor(public side: number) {}
  area() {
    return this.side ** 2;
  }
}

class AreaCalculator {
  calculate(shape: Shape) {
    return shape.area();
  }
}
```

---

### 2.3 Liskov Substitution Principle (LSP)

- **Ý nghĩa:** Class con có thể thay thế hoàn toàn cho class cha mà không làm hỏng chương trình.
- **Hiểu đơn giản:** Nếu dùng class con mà hệ thống vẫn chạy ổn định như class cha thì mới đúng.
- **Ví dụ:** Chim sẻ có thể bay nên kế thừa "chim biết bay", trong khi chim cánh cụt chỉ nên kế thừa "chim" chứ không nên ép vào nhóm "chim bay".

```ts
// ❌ Sai: Penguin không bay được nhưng kế thừa Bird với fly()
class Bird {
  fly() {
    console.log("I can fly!");
  }
}
class Penguin extends Bird {
  fly() {
    throw new Error("Penguin can't fly!");
  }
}

// ✅ Đúng: Tách abstraction phù hợp
interface Bird {
  eat(): void;
}

interface FlyingBird extends Bird {
  fly(): void;
}

class Sparrow implements FlyingBird {
  eat() {
    console.log("Sparrow eating");
  }
  fly() {
    console.log("Sparrow flying");
  }
}

class Penguin implements Bird {
  eat() {
    console.log("Penguin eating");
  }
}
```

---

### 2.4 Interface Segregation Principle (ISP)

- **Ý nghĩa:** Interface không nên quá "béo", class không nên bị buộc implement những thứ không dùng.
- **Hiểu đơn giản:** Chia nhỏ interface cho từng nhu cầu cụ thể.
- **Ví dụ:** Máy in chỉ cần implement `IPrinter`, không cần phải gánh thêm `IScanner` hay `IFax`.

```ts
// ❌ Sai: Class buộc phải implement cả method không cần
interface IMachine {
  print(): void;
  scan(): void;
  fax(): void;
}

class OldPrinter implements IMachine {
  print() {
    console.log("Printing...");
  }
  scan() {
    throw new Error("Not supported");
  }
  fax() {
    throw new Error("Not supported");
  }
}

// ✅ Đúng: Chia nhỏ interface
interface IPrinter {
  print(): void;
}
interface IScanner {
  scan(): void;
}
interface IFax {
  fax(): void;
}

class SimplePrinter implements IPrinter {
  print() {
    console.log("Printing...");
  }
}

class MultiFunctionPrinter implements IPrinter, IScanner, IFax {
  print() {
    console.log("Printing...");
  }
  scan() {
    console.log("Scanning...");
  }
  fax() {
    console.log("Faxing...");
  }
}
```

---

### 2.5 Dependency Inversion Principle (DIP)

- **Ý nghĩa:** Code nên phụ thuộc vào abstraction (interface) thay vì implementation (class cụ thể).
- **Hiểu đơn giản:** Module cấp cao không nên phụ thuộc trực tiếp module cấp thấp, cả hai nên phụ thuộc vào abstraction.
- **Ví dụ:** `ReportService` gọi `IReportRepository` thay vì fix cứng vào `MySQLReportRepository`.

```ts
// ❌ Sai: Service phụ thuộc trực tiếp vào DB cụ thể
class MySQLReportRepository {
  getReports() {
    return ["report from MySQL"];
  }
}

class ReportService {
  constructor(private repo: MySQLReportRepository) {}
  showReports() {
    return this.repo.getReports();
  }
}

// ✅ Đúng: Phụ thuộc abstraction (interface)
interface ReportRepository {
  getReports(): string[];
}

class MySQLReportRepository implements ReportRepository {
  getReports() {
    return ["report from MySQL"];
  }
}

class MongoReportRepository implements ReportRepository {
  getReports() {
    return ["report from MongoDB"];
  }
}

class ReportService {
  constructor(private repo: ReportRepository) {}
  showReports() {
    return this.repo.getReports();
  }
}

// Sử dụng
const service = new ReportService(new MongoReportRepository());
console.log(service.showReports());
```

---

## 3. Những điều lưu ý để tránh hiểu sai về SOLID.

Khi nhìn lại quy tắc **SRP**, có thể thấy trong NestJS (và nhiều framework khác), mỗi service (ví dụ: `UserService`, `PostService`) thường gom hết CRUD (create, read, update, delete) trong một class duy nhất. Nhìn qua có vẻ vi phạm SRP, nhưng thực tế:

- NestJS service bản chất là **application service**, gom logic xử lý request liên quan đến một entity/domain.
- Việc có `createUser`, `updateUser`, `deleteUser`, `getUser` trong cùng `UserService` vẫn được xem là **một trách nhiệm** - đó là quản lý User.
- Nếu tách nhỏ thành `UserCreationService`, `UserDeletionService`, `UserQueryService` thì số lượng file/service tăng rất nhiều, gây **over-engineering** cho dự án.
- **SOLID là nguyên tắc, không phải luật cứng nhắc.**
- Trong dự án thực tế, người ta trade-off để code **dễ đọc, dễ maintain** quan trọng hơn việc "theo đúng giáo trình".

> Nói cách khác: không phải là không theo SOLID, mà là áp dụng nó ở mức thực dụng, tránh over-engineering.

---

Ngày đã dần xuống, bài viết cũng dài. Đến đây, chúc cho vị đại hiệp đọc xong bí kíp có thể vận dụng thành thạo, tung hoành giang hồ lập trình, khiến người đời hân hoan nể phục. **Cáo từ.**
