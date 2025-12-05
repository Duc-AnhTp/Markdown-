## B.2. Truy vấn & Nghiệp vụ (Logic)
### B.2.1. Import 1 triệu dữ liệu từ Người A vào MongoDB  

Dữ liệu được Người A xuất ra dưới dạng file JSON (ví dụ: orders_1m.json) chứa khoảng 1.000.000 document.  

Trên máy cài đặt MongoDB, tôi sử dụng công cụ dòng lệnh mongoimport để import dữ liệu vào database btl_bigdata, collection orders.

Câu lệnh thực hiện như sau: 
```javascript
mongoimport \
  --uri="mongodb://localhost:27017" \
  --db=btl_bigdata \
  --collection=orders \
  --file=orders_1m.json \
  --jsonArray
```
Sau khi import, kiểm tra nhanh trong mongosh:

```javascript
mongosh "mongodb://localhost:27017"
use btl_bigdata
db.orders.countDocuments()
```
Kết quả trả về xấp xỉ 1.000.000 document, đảm bảo dữ liệu đầu vào đủ lớn để thử nghiệm hiệu năng truy vấn (đặc biệt cho phần so sánh trước/sau khi tạo index ở mục B.3).  

### B.2.2. Chuyển đổi Q1–Q3 từ SQL sang MQL

Từ bài toán SQL ban đầu (trên mô hình quan hệ), nhóm yêu cầu giữ nguyên logic nghiệp vụ nhưng viết lại bằng ngôn ngữ truy vấn của MongoDB (MQL) trên mô hình embedded orders. 

Tại thời điểm này chưa tạo bất kỳ index nào, mục tiêu là quan sát “hiệu năng thô” của 3 câu truy vấn chính.

Database sử dụng: btl_bigdata  

Collection: orders  

```javascript
use btl_bigdata
```
#### Q1 – Lọc đơn hàng theo trạng thái, sắp xếp theo ngày, lấy 10 bản ghi

Nghiệp vụ: Lấy các đơn hàng có trạng thái “Đang giao” hoặc “Hoàn thành”, ưu tiên đơn mới nhất, giới hạn số lượng kết quả để hiển thị.

MQL tương ứng:

```javascipt
db.orders.find(
  { status: { $in: ["Đang giao", "Hoàn thành"] } },   // điều kiện lọc
  {
    order_code: 1,
    order_date: 1,
    status: 1,
    total_amount: 1,
    "customer.name": 1
  }                                                   // các field cần hiển thị
)
.sort({ order_date: -1 })                             // sắp xếp đơn mới nhất trước
.limit(10);                                           // giới hạn 10 bản ghi
```  

#### Q2 – Thống kê doanh thu và số lượng đơn theo từng ngày

Nghiệp vụ: Thực hiện thống kê tổng doanh thu (total_amount) và số lượng đơn hàng theo từng ngày, phục vụ mục đích phân tích kinh doanh theo thời gian.

MQL sử dụng Aggregation Pipeline:

```javascript
db.orders.aggregate([
  {
    $group: {
      _id: {
        year:  { $year: "$order_date" },
        month: { $month: "$order_date" },
        day:   { $dayOfMonth: "$order_date" }
      },
      totalRevenue: { $sum: "$total_amount" },
      orderCount:   { $sum: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      date: {
        $dateFromParts: {
          year:  "$_id.year",
          month: "$_id.month",
          day:   "$_id.day"
        }
      },
      totalRevenue: 1,
      orderCount: 1
    }
  },
  { $sort: { date: 1 } }   // sắp xếp tăng dần theo ngày
]);
```

#### Q3 – Top 10 sản phẩm bán chạy nhất (từ mảng embedded items)

Nghiệp vụ: Tìm danh sách các sản phẩm được mua nhiều nhất trong toàn bộ 1.000.000 đơn hàng, dựa trên tổng số lượng (quantity) và tính kèm tổng doanh thu mang lại cho từng sản phẩm.

Mô hình embedded lưu sản phẩm trong mảng items của từng document orders, do đó cần sử dụng $unwind để “dỡ” mảng thành từng dòng, sau đó group theo sản phẩm:

```javascript
db.orders.aggregate([
  { $unwind: "$items" },   // tách từng item trong mảng items thành dòng riêng
  {
    $group: {
      _id: {
        product_id:   "$items.product_id",
        product_name: "$items.product_name"
      },
      totalQuantity: { $sum: "$items.quantity" },
      totalRevenue:  {
        $sum: { $multiply: ["$items.quantity", "$items.price"] }
      },
      orderCount:    { $sum: 1 }   // số đơn có chứa sản phẩm này
    }
  },
  { $sort: { totalQuantity: -1 } },  // sản phẩm bán chạy nhất (số lượng nhiều nhất) lên đầu
  { $limit: 10 },
  {
    $project: {
      _id: 0,
      product_id:   "$_id.product_id",
      product_name: "$_id.product_name",
      totalQuantity: 1,
      totalRevenue: 1,
      orderCount: 1
    }
  }
]);
```





### B.2.3. Thử nghiệm tính linh hoạt: thêm trường mới hasPromotion

Để minh họa tính linh hoạt của mô hình NoSQL (schema-less), nhóm chọn thử nghiệm với một trường mới là hasPromotion. Trường này biểu diễn việc đơn hàng có áp dụng khuyến mãi hay không.

Điểm quan trọng là:

- Không cần thay đổi cấu trúc schema ở mức hệ thống (không có ALTER TABLE như trong SQL).

- Có thể thêm trực tiếp trường mới vào một phần dữ liệu hiện có, chỉ bằng các câu lệnh update.

Ví dụ, gán hasPromotion = true cho các đơn hàng có tổng tiền lớn hơn 2.000.000:

```javascript
db.orders.updateMany(
  { total_amount: { $gt: 2000000 } },
  { $set: { hasPromotion: true } }
);
```

Sau khi cập nhật, có thể thực hiện một số truy vấn kiểm tra:

- Đếm số đơn hàng có khuyến mãi:
  
  ```javascript
  db.orders.countDocuments({ hasPromotion: true });
  ```
  
- Lấy 10 đơn hàng có khuyến mãi, sắp xếp theo ngày đặt hàng:
  ```javascript
  db.orders.find(
    { hasPromotion: true },
    {
      order_code: 1,
      order_date: 1,
      total_amount: 1,
      hasPromotion: 1
    }
  )
  .sort({ order_date: -1 })
  .limit(10);
  ``` 
- Thống kê tổng doanh thu của các đơn hàng có khuyến mãi:
  ```javascript
  db.orders.aggregate([
    { $match: { hasPromotion: true } },
    {
      $group: {
        _id: null,
        totalRevenuePromotion: { $sum: "$total_amount" },
        orderCountPromotion:   { $sum: 1 }
      }
    }
  ]);
  ```

Qua thử nghiệm này, có thể thấy việc thêm trường mới trong MongoDB rất linh hoạt, không ảnh hưởng đến các document không có trường đó (chúng vẫn hợp lệ) và các truy vấn có thể vừa lọc theo hasPromotion, vừa kết hợp với các điều kiện khác như status, order_date nếu cần.




### B.2.4. Ghi nhận kết quả và bàn giao cho Người D

Sau khi hoàn thành các bước trên, tôi đã:

- Chuẩn hóa lại toàn bộ câu truy vấn MQL cho Q1, Q2, Q3 và các câu truy vấn liên quan đến trường mới hasPromotion dưới dạng một file script (có thể đặt tên queries_no_index.mongo) với đầy đủ comment giải thích.

- Chạy thử Q1–Q3 trên dữ liệu 1.000.000 document trước khi tạo index, đồng thời sử dụng:

  ```javascript
  .explain("executionStats")
  ```
  để ghi lại:
  - executionTimeMillis

  - totalDocsExamined

  - Các thông tin liên quan đến cách MongoDB thực hiện truy vấn.

- Bàn giao các script truy vấn và số liệu thực nghiệm này cho Người D, để dùng làm cơ sở so sánh hiệu năng trước và sau khi áp dụng index trong phần tối ưu truy vấn (mục B.3 của báo cáo).
