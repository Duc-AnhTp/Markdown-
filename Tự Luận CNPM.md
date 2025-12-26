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

