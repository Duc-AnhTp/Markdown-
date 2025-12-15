### 1) Document model đề xuất (match Q1/Q2/Q3)

Collections

- products (reference → suppliers)

- suppliers

- orders (embedded items như OrderDetails, reference → products.productId)

- inventoryLogs (reference → products.productId)

Products

```javascript
{
  productId: 1234,
  productName: "A",
  district: "District1",
  expiryDate: ISODate("2026-01-10"),
  stockQuantity: 100,
  price: 12000,
  supplierId: 7            // reference suppliers.supplierId
}
```

suppliers
```javascript
{ supplierId: 7, supplierName: "Supplier 7", phone: "...", email: "..." }
```

orders (embedded items ≈ OrderDetails)
```javascript
{
  orderId: 9001,
  status: "Completed",
  orderDate: ISODate("2025-11-01"),
  items: [
    { productId: 1234, quantity: 2, unitPrice: 12000 },
    { productId: 5555, quantity: 1, unitPrice: 7000 }
  ]
}
```

inventoryLogs
```javascript
{ productId: 1234, logDate: ISODate("2025-12-10"), changeAmount: -3 }
```

#### Embedded ở orders.items để đọc/ghi đơn hàng nhanh; Reference ở products→suppliers, inventoryLogs→products để tránh lặp dữ liệu và dễ cập nhật.

---

### 2) Q1 MongoDB (lọc district + sort expiryDate + limit 20)

MySQL gốc: Products lọc District IN, sort ExpiryDate, limit 20 
```javascript
db.products
  .find(
    { district: { $in: ["District1", "District2"] } },
    { _id: 0, productId: 1, productName: 1, district: 1, expiryDate: 1, stockQuantity: 1, price: 1 }
  )
  .sort({ expiryDate: 1 })
  .limit(20);
```

Index (tương đương composite MySQL District+ExpiryDate) 
```javascript
db.products.createIndex({ district: 1, expiryDate: 1 });
```

Explain
```javascript
db.products
  .find({ district: { $in: ["District1", "District2"] } })
  .sort({ expiryDate: 1 })
  .limit(20)
  .explain("executionStats");
```
---

### 3) Q2 MongoDB (group theo tháng 12 tháng gần nhất)

MySQL gốc: group theo DATE_FORMAT(LogDate,'%Y-%m'), lọc LogDate >= NOW()-12 months 
```javascript
const from = new Date();
from.setMonth(from.getMonth() - 12);

db.inventoryLogs.aggregate([
  { $match: { logDate: { $gte: from } } },
  {
    $group: {
      _id: { $dateToString: { format: "%Y-%m", date: "$logDate" } },
      total_change: { $sum: "$changeAmount" },
      total_events: { $sum: 1 }
    }
  },
  { $sort: { _id: 1 } }
]);
```

Index (tương đương index theo thời gian LogDate) 
```javascript
db.inventoryLogs.createIndex({ logDate: 1 });
```

Explain
```javascript
db.inventoryLogs.explain("executionStats").aggregate([
  { $match: { logDate: { $gte: from } } },
  {
    $group: {
      _id: { $dateToString: { format: "%Y-%m", date: "$logDate" } },
      total_change: { $sum: "$changeAmount" },
      total_events: { $sum: 1 }
    }
  },
  { $sort: { _id: 1 } }
]);
```

---
4) Q3 MongoDB (Top sản phẩm theo doanh thu + supplierName)

MySQL gốc: join Orders + OrderDetails + Products + Suppliers, lọc o.Status='Completed', group + sort revenue desc limit 10 

**Phiên bản “NoSQL đúng chất” (orders có embedded items)**
```javascript
db.orders.aggregate([
  { $match: { status: "Completed" } },
  { $unwind: "$items" },

  {
    $group: {
      _id: "$items.productId",
      revenue: { $sum: { $multiply: ["$items.quantity", "$items.unitPrice"] } },
      total_qty: { $sum: "$items.quantity" }
    }
  },

  { $sort: { revenue: -1 } },
  { $limit: 10 },

  // join sang products để lấy ProductName + SupplierID
  {
    $lookup: {
      from: "products",
      localField: "_id",
      foreignField: "productId",
      as: "p"
    }
  },
  { $unwind: "$p" },

  // join sang suppliers để lấy SupplierName
  {
    $lookup: {
      from: "suppliers",
      localField: "p.supplierId",
      foreignField: "supplierId",
      as: "s"
    }
  },
  { $unwind: { path: "$s", preserveNullAndEmptyArrays: true } },

  {
    $project: {
      _id: 0,
      productId: "$p.productId",
      productName: "$p.productName",
      supplierName: "$s.supplierName",
      revenue: 1,
      total_qty: 1
    }
  }
]);
```

Index gợi ý
```javascript
db.orders.createIndex({ status: 1 });
db.orders.createIndex({ "items.productId": 1 });     // multikey
db.products.createIndex({ productId: 1 });           // cho $lookup
db.products.createIndex({ supplierId: 1 });
db.suppliers.createIndex({ supplierId: 1 });


Explain

db.orders.explain("executionStats").aggregate([
  { $match: { status: "Completed" } },
  { $unwind: "$items" },
  {
    $group: {
      _id: "$items.productId",
      revenue: { $sum: { $multiply: ["$items.quantity", "$items.unitPrice"] } },
      total_qty: { $sum: "$items.quantity" }
    }
  },
  { $sort: { revenue: -1 } },
  { $limit: 10 }
]);
```

Lưu ý: nếu data bạn đang generate dùng status tiếng Việt "Hoàn thành" (như script faker của bạn) 

 thì đổi { status: "Completed" } thành { status: "Hoàn thành" }.
