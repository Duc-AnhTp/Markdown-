### 1) Yêu cầu phần mềm bao gồm những gì?

Yêu cầu phần mềm (software requirements) thường gồm các nhóm chính sau:

A. Yêu cầu chức năng (Functional Requirements)
- Phần mềm phải làm gì: các chức năng, nghiệp vụ, xử lý dữ liệu.
  - Ví dụ: đăng nhập, tìm kiếm, tạo đơn hàng, tính tiền, xuất báo cáo,…

B. Yêu cầu phi chức năng (Non-functional Requirements)
- Phần mềm làm tốt đến mức nào: chất lượng và thuộc tính vận hành:
  Hay gặp:
  - Hiệu năng (thời gian phản hồi, số người dùng đồng thời)
  - Bảo mật (phân quyền, mã hóa, audit log)
  - Độ tin cậy/sẵn sàng (uptime, backup, recovery)
  - Tính dễ dùng (UI/UX, accessibility)
  - Khả năng mở rộng, bảo trì, tương thích
  - Tuân thủ (quy định, tiêu chuẩn)

C. Ràng buộc (Constraints)
- Những “giới hạn bắt buộc” khi xây dựng hệ thống:
  - Công nghệ phải dùng (VD: .NET/Java), nền tảng (Windows/Linux)
  - Ngân sách, thời gian, nhân lực
  - Quy định pháp lý, chính sách công ty
    
D. Yêu cầu dữ liệu
- Dữ liệu vào/ra, cấu trúc dữ liệu, lưu trữ, vòng đời dữ liệu.
  - Ví dụ: thông tin khách hàng gồm các trường nào, dữ liệu lưu bao lâu,…
    
E. Yêu cầu giao diện & tích hợp (Interfaces)
- Giao diện người dùng (UI), API, tích hợp hệ thống khác, thiết bị phần cứng.
  - Ví dụ: tích hợp cổng thanh toán, ERP, gửi email/SMS,…

F. Yêu cầu về môi trường vận hành (Operational/Deployment)
- Hạ tầng chạy ở đâu, cấu hình, mạng, logging/monitoring.
Ví dụ: chạy trên cloud, cần autoscaling, log tập trung,…

Tóm lại: FR + NFR + Constraints + Data + Interfaces + Operational là bộ khung đầy đủ nhất.


### 2) So sánh “Xác định yêu cầu” và “Đặc tả yêu cầu”
| Tiêu chí             | Xác định yêu cầu (Requirements Definition / Elicitation & Analysis)                      | Đặc tả yêu cầu (Requirements Specification – SRS)                                         |
| -------------------- | ---------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Mục tiêu             | **Tìm hiểu – làm rõ – thống nhất** nhu cầu                                               | **Ghi lại chính thức** yêu cầu để làm căn cứ thiết kế/thi công/kiểm thử                   |
| Bản chất             | Quá trình trao đổi, phân tích, ưu tiên, xử lý mâu thuẫn                                  | Tài liệu hóa có cấu trúc, rõ ràng, kiểm chứng được                                        |
| Đầu vào              | Mong muốn của stakeholder, bối cảnh nghiệp vụ, quy trình hiện tại                        | Kết quả đã được làm rõ từ bước xác định yêu cầu                                           |
| Đầu ra               | Danh sách yêu cầu đã hiểu đúng, mô hình nghiệp vụ, user stories/backlog, use case sơ bộ… | Tài liệu SRS / đặc tả: mô tả chi tiết FR/NFR, interface, constraints, tiêu chí chấp nhận… |
| Mức độ chi tiết      | Có thể còn “mở”, ưu tiên và phạm vi có thể điều chỉnh                                    | **Cụ thể, không mơ hồ**, có thể kiểm thử/đo lường                                         |
| Người tham gia chính | BA/PO + khách hàng + dev/QA (để làm rõ)                                                  | BA/PO chủ trì, dev/QA dùng để triển khai & test, stakeholder ký duyệt                     |
| Khi nào dùng mạnh    | Giai đoạn đầu và lặp lại khi có thay đổi                                                 | Khi cần “chốt” phiên bản yêu cầu, làm baseline, bàn giao cho dev/test                     |


### 3) Khi nào chúng ta có thể áp dụng mô hình quy trình kỹ nghệ phần mềm theo Agile?
Chúng ta có thể áp dụng mô hình quy trình kỹ nghệ phần mềm theo Agile khi dự án cần tính linh hoạt cao và có thể phát triển theo từng phần nhỏ (increment) trong các vòng lặp ngắn (Sprint). Cụ thể, Agile phù hợp nhất trong các trường hợp sau:

- Yêu cầu chưa rõ ràng hoặc thay đổi thường xuyên: Khách hàng chưa xác định đầy đủ nhu cầu ngay từ đầu, hoặc môi trường/ thị trường biến động nhanh nên cần thích ứng liên tục.

- Cần đưa sản phẩm ra sớm (time-to-market): Doanh nghiệp muốn phát hành sớm các chức năng quan trọng để lấy phản hồi hoặc tạo lợi thế cạnh tranh.

- Khách hàng/ Product Owner tham gia tích cực: Có người đại diện nghiệp vụ làm việc thường xuyên với nhóm phát triển để phản hồi, ưu tiên backlog và nghiệm thu từng increment.

- Đội ngũ phát triển có năng lực và phối hợp tốt: Nhóm nhỏ–vừa, giao tiếp hiệu quả, có khả năng tự tổ chức và ra quyết định nhanh.

- Dự án có độ phức tạp và rủi ro cao: Làm theo các vòng lặp ngắn giúp phát hiện vấn đề sớm, giảm rủi ro kỹ thuật và điều chỉnh kịp thời.

- Có khả năng kiểm thử và tích hợp thường xuyên: Mỗi Sprint tạo ra một phiên bản chạy được, dễ demo và đánh giá chất lượng.

- Ngược lại, Agile thường kém phù hợp nếu yêu cầu đã cố định chặt ngay từ đầu, ít thay đổi, hoặc dự án chịu ràng buộc nặng về quy trình phê duyệt/tài liệu và không thể nhận phản hồi thường xuyên.
