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
   
---
### 8) YARN trong Hadoop có vai trò gì?
YARN là lớp quản lý tài nguyên cụm (CPU core, RAM) và cấp phát tài nguyên cho ứng dụng.
  - YARN cấp phát tài nuyên phù hợp theo yêu cầu ứng dụng
  - Có từ Hadoop 2.0
  - Cho phép MapReduce và non-MapReduce chạy chung một cụm
  - Với MapReduce, vai trò job tracker được thực hiện bởi "application tracker" (hiểu theo hướng AM)

YARN (Yet Another Resource Negotiator) tách quản lý tài nguyên khỏi lập lịch ứng dụng:
  - RessourceManager (RM): quản lý tài nguyên toàn cụm (CPU/memory) + scheduler
  - NodeManager (NM): agent trên mỗi node, quản lý container, báo cáo tình trạng node
  - ApplicationMaster (AM): "master per application", đàm phán container với RM và điều phối task của ứng dụng

Lợi ích:
  - Chạy nhiều framework khác nhau trên Hadoop (MapReduce, Spark, Tez, ...)
  - Tăng khả năng mở rộng, giảm bottleneck kiểu JobTracker

---
### 9) Hadoop giải quyết bài toán chịu lỗi như thế nào?
Chịu lỗi ở cả lưu trữ và tính toán:
- Lưu trữ (HDFS):
  - Replication block (thường 3)
  - Tự phát hiện DN chêt qua heartbeat
  - Tự tái nhân bản (re-replication)
  - Checksum chống hỏng dữ liệu
- Tính toán (MapReduce/YARN)
  - Retry task: task fail -> chạy lại ở node khác
  - Speculative execution: nếu có task chạy "lụt" (straggler) -> chạy bản sao ở node khác, lấy kết quả về trước
  - Outpt trung gian lưu ra disk (MapReduce) nên có thể phục hồi theo stage
  - Với YARN : nếu container chết --> AM có thể xin container khác và chạy lại

---
### 10) Kiến trúc của HDFS?
Thành phần chính:
- NameNode: quản lý metadata:
  - namespace (cây thư mục, file)
  - mapping file -> list block
  - mapping block -> DataNode locations
- DataNode: lưu trữ dữ liệu block trên local disk, phục vụ read/write
- Client: truy cập HDFS; dữ liệu đi trực tiếp client <-> DataNode
- (Tùy cấu hình) JournalNodes/ZKFC/Standby NameNode cho HA
- (Cũ) SecondaryNameNode/Checkpoint Node

---
### 11) Nguyên lý thiết kế cốt lõi của HDFS?
- Tối ưu cho file lớn, streaming read/write
- Write once, read many (truyền thống; hiện nay hỗ trợ append hạn chế nhưng vẫn ưu tiên mô hình này)
- Tối ưu throughput hơn là latency thấp (không giống hệ thống file cho OLTP)
- Dữ liệu được chia block lớn để giảm overhead metadata
- Chịu lỗi bằng replication trên phần cứng phổ thông
- Data locality: ưu tiên chạy compute gần data

---
### 12) Mô thức xử lý dữ liệu MapReduce?
MapReduce là mô hình batch gồm các phase:
1. Map phase
   - Input split -> nhiều mapper
   - Mapper đọc record (k, v) -> phát ra (k2, v2)
2. Shuffle + Sort
   - Framework partition theo key (partitioner)
   - Kéo dữ liệu qua network (shuffle)
   - Sort/Group theo key để chuẩn bị reduce
3. Reduce phase
   - Reduce nhận (key, list(values)) -> tạo output
  
Thành phần hay gặp:
- Combiner: "mini-reduce" phía map để giảm shuffle
- Partitoner: quyết định key nào sang reducer nào
- InputFormat/OutputFormat: cách đọc/ghi dữ liệu

---
### 13) Các cơ sở dữ liệu truyền thống mở rộng bằng cách nào?
Các cơ sở dữ liệu truyền thống thường được mở rộng theo:
- Scale-up (vertical): nâng CPU/RAM/SSD cho 1 máy -> giới hạn bởi phần cứng, chi phí cao
- Read scaling bằng replication:
  - 1 primary (write), nhiều read replicas
  - phù hợp workload đọc nhiều
- Partitioning/Sharding (scale-out):
  - chia dữ liệu theo key (range/hash) ra nhiều node
  - phức tạp hơn vì join/transaction cross-shard
- Clustering (shared-nothing/shared-disk tùy sản phẩm)
- Caching (Redis/memcached) để giảm tải DB
   
---
### 14) Data sharding là gì?
Shardign = chia 1 bảng/collection thành nhiều phần (shard) đặt trên nhiều node, mỗi shard giữ một tập con dữ liệu.
- Theo hash (hash(user_id) -> shard)
- Theo range (A-F shard1, G-L shard2...)
- Theo directory/lookup (bảng định tuyến)

Mục tiêu:
- Tăng dung lượng và throughput bằng scale-out
- Nhưng làm tăng độ phức tạp: routing, rebalance, query cross-shard, transaction

---
### 15) Data replicating là gì?
Replicating = sao chép dữ liệu từ node này sang node khác để:
- Tăng tính sẵn sàng (HA)
- Tăng khả năng đọc (read scale)
- Giảm rủi ro mất dữ liệu

Các dạng:
- Synchronous (đợi replica ack mới commit) -> mạnh về consistency, tăng latency
- Asynchoronous (commit trước, replicate sau) -> nhanh hơn, nhưng có thể lag (eventual consistency)

---
### 16) Two-Phase Commit Protocol là gì?
2PC là giao thức commit phân tán đảm bảo "atomic commit" cho transaction trên nhiều participant.

Coordinator (điều phối) + nhiều Participants

Phase 1: Prepare / Voting
1. Coordinator gửi prepare
2. Participants ghi log, khóa tài nguyên, kiểm tra có commit được không
3. Trả vote-commit hoặc vote-abort

Phase 2: Commit / Abort
- Nếu tất cả vote-commit -> coordinator gửi commit
- Nếu có ai abort -> coordinator gửi abort

Nhược điểm lớn:
- Blocking: nếu coodinator chết đúng lúc, participants có thể vị kẹt (giữ lock)
- Tốn chi phí log + roound-trip network

---
### 17) CAP Theorem là gì?
CAP nói rằng trong hệ thống phân tán, khi có network partition (P), bạn phải chọn ưu tiên giữa:
- C (Consistency): mọi node thấy cùng một dữ liệu tại cùng thời điểm (thường hiều là linearizability)
- A (Availability): mọi request nhận được response (không lỗi / timeout) từ node còn sống
- P (Partition Tolerance): hệ thống vẫn hoạt động dù mạng bị chia cắt

Trong thực tế:
- Partition có thể xảy ra -> đa số hệ thống phải chấp nhận "P"
- Khi xảy ra partition:
  - Chọn CP: giữ consistency, có thể từ chối một số request
  - Chọn AP: luôn trả lời, chấp nhận dữ liệu tạm thời lệch và hòa giải sau


---
### 18) ACID trong cơ sở dữ liệu quan hệ là gi?
ACID là 4 tính chất transaction:
- Atomicity: hoặc tất cả thành công hoặc tất cả rollback
- Consistency: transaction đưa DB từ trạng thái hợp lệ -> hợp lệ (ràng buộc/constraint không bị phá)
- Isolation: các transaction song song không "dẫm" lên nhau (mức isolation: Read Commited, Repeatable Read, ...)
- Durability: đã commit thì không mất (nhờ WAL, fsync, replication,...)


---
### 19) Trading-Off Consistency là gì?
"Trade-off consistency" = chấp nhận giảm mức nhất quán để đổi lấy:
- Latency thấp hơn
- Availability cao hơn
- Throughtput tốt hơn
- Vận hành đơn giản hơn trong môi trường phân tán

Ví dụ các mức/chiến lược:
- Strong consistency (linearizable) <-> eventual consistency
- Quorum reads/writes (N, R, W): tăng R/W để mạnh consistency hơn nhưng tốn latency
- Read-your-writes / Monotonic reads: các guarantee trung gian (session consistency)

---
### 20) BASE Properties trong hệ phân tán là gì?
BASE là đối trọng "mềm" của ACID trong nhiều hệ NoSQL:
- Basically Available: hệ thống cố gắng luôn trả lời
- Soft state: trạng thái có thể thay đổi theo thời gian ngay cả khi không có input (do replicatoin async, reconciliation...)
- Eventual consistency: nếu không có update mới, cuối cùng các replica sẽ hội tụ về cùng một giá trị

---
### 21) Eventual Consistency là gì?
Eventual Consistency: khi không còn cập nhật mới, sau một thời gian, tất cả replica sẽ trở nên nhất quán.
- Có thể đọc thấy dữ liệu "cũ" trong thời gian ngắn
- Thường đi kèm cơ chế hòa giải:
  - last-write-wins (LWW)
  - vector clocks/versioning
  - conflict resolution ở app
 

---
### 22) Sự khác nhau giữa Spark và MapReduce
- Mô hình thực thi
  - MapReduce: pipeline cứng Map -> Shuffle/Sort -> Reduce, ghi trung gian ra disk nhiều
  - Spark: DAG engine (chuỗi stage linh hoạt), tối ưu và pipelining tốt hơn
- In-memory
  - Spark giữ dữ liệu trong RAM (cache/persist) -> rất mạnh cho iterative/interactive
  - MapReduce chủ yếu disk-based
- API
  - Spark: RDD/DataFrame/SQL/MLib/Streaming
  - MapReduce: API thấp hơn, code dài, ít linh hoạt
- Use case
  - MapReduce: batch lớn, đơn giản, chấp nhận latency cao
  - Spark: ETL, interactive analytics, iterative ML, near-real-time streaming
 

---
### 23) So sánh hiệu năng Spark và MapReduce
Thông thường:
- Spark nhanh hơn đáng kể (có thể nhiều lần đến hàng chục lần) cho:
  - thuật toán lặp (iterative) như ML, graph
  - interactive query/ETL nhiều bước vì giảm I/O disk
  - workload cần reuse dataset (cache)
- MapReduce có thể “ổn” hoặc cạnh tranh khi:
  - job 1 lần, tuyến tính, dữ liệu cực lớn mà RAM không đủ
  - môi trường ưu tiên đơn giản/ổn định, ít tuning
Hiệu năng thực tế phụ thuộc:
- RAM/GC, số partition, shuffle, skew, serialization, cấu hình cluster,...

---
### 24) Resilient Distributed Dataset (RDD) là gì?
RDD là cấu trúc dữ liệu cốt lõi của Spark:
- Distributed: dữ liệu chia thành nhiều partition trên cluster
- Immutable: transformation tạo RDD mới
- Resilient: chịu lỗi nhờ lineage (lịch sử biến đổi). Khi mất partition → Spark tính lại từ nguồn/cha (thay vì phải replicate mọi thứ)
- Hỗ trợ 2 dạng dependency:
  - Narrow (map/filter): dễ pipeline
  - Wide (groupByKey/join): cần shuffle



