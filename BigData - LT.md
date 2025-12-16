## Đề cương Nhập môn Dữ liệu lớn
---
### 1) Hadoop giải quyết bài toán mở rộng như nào?

Hadoop giải quyết mở rộng theo *chiều ngang (scale-out)*: thay vì nâng cấp 1 máy thật mạnh, ta thêm nhiều node rẻ hơn vào cụm để tăng lưu trữ & tăng năng lực xử lý song song.

  - *Lưu trữ (HDFS):* file đucợ chia thành các chunks lớn (ví dụ 64MB) và phân tán lên nhiều DataNode, giúp giảm metadata và giảm chi phí truyền dữ liệu
  - *Tính toán (MapReduce/YARN):* 1 job được phân rã thành nhiều task độc lập chạy trên nhiều node để đạt khả mở.
  - *Tận dụng "data locality":* khi chạy MapReduce thường "đẩy code tới nơi có dữ liệu" để giảm truyền dữ liệu qua mạng.

---
### 2) Mô tả cách thức 1 client đọc dữ liệu trên HDFS?
Luồng đọc tiêu biểu: 
1. Client hỏi NameNode metadata: file gồm những block/chunk nào, mỗi block đang nằm trên các DataNode nào (NameNode quản lý ánh xạ "tệp -> vị trí chunks").
2. Chọn replica gần nhất để đọc (ưu tiên cùng naode / cùng rack) nhằm tối ưu tốc độ.
3. Client đọc trực tiếp từ DataNode (NameNode không "đứng giữa" dòng dữ liệu).
4. Kiểm tra toàn vẹn bằng checksum: client nhận cả data + checksum; nếu checksum sai thì client thử đọc từ bản nhân bản khác.

---
### 3) Mô tả cách thức 1 client ghi dữ liệu trên HDFS?

Luồng ghi tiêu biểu:
1. Client xin NameNode danh sách DataNode để đặt replicas cho block mới.
2. Ghi theo pipeline (data pipelining): client gửi dữ liệu tới DataNode #1; DataNode #1 chuyển tiếp sang DataNode #2; … cho đến đủ số bản sao.
3. ACK ngược pipeline: khi các replica đã ghi xong, xác nhận ngược về client; client chuyển sang block tiếp theo.
4. HDFS có mô thức append (ghi thêm) giúp giảm chi phí điều khiển tương tranh.

---
### 4) Cơ chế chịu lỗi của DataNode trong HDFS?
HDFS phát hiện và xử lý lỗi DataNode chủ yếu nhờ:

- Heartbeat: DataNode gửi heartbeat định kỳ về NameNode (slide nêu 3s/lần); NameNode dùng heartbeat để phát hiện bất thường.
- Block Report: DataNode định kỳ báo cáo các block/chunk đang giữ.
- Khi NameNode phát hiện node lỗi/không truy cập được → chọn DataNode mới để tái nhân bản nhằm đưa hệ về mức replica mong muốn.

---
### 5) Cơ chế nhân bản dữ liệu trong HDFS?
- Mỗi chunk thường được sao làm 3 bản (replication factor mặc định hay gặp).
- Chiến lược đặt chỗ (rack-aware) điển hình: 1 replica trên máy đích, replica #2 khác rack, replica #3 cùng rack với replica #2; các replica khác có thể đặt ngẫu nhiên.
- Client đọc replica gần nhất để tối ưu.

---
### 6) HDFS giải quyết single-point-of-failure cho NameNode bằng cách nào?
## ...

---
### 7) Vai trò của JobTracker và TaskTracker trong Hadoop
(Đây là kiến trúc MapReduce"classic" - Hadoop 1.x)
  - JobTracker (master):
    - Nhận job từ clientl, chia job thành các map/reduce tasks.
    - Lập trình chạy task (ưu tiên data locality), theo dõi tiến độ, xử lý retry khi task fail.
    - Quản lý "tài nguyên kiểu slot" (map slots/reduce slots) của cluster.
  - TaskTracker (worker):
    - Chạy trên mỗi node, thực thi tasks do JobTracker giao.
    - Gửi heartbeat/trạng thái task về JobTracker.
    - Quanrlys các process JVM chạy task trên node.
