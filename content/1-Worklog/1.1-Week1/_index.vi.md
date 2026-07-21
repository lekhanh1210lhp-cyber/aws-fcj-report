---
title: "Worklog Tuần 1"
date: "2025-09-09"
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu tuần 1:

- Phân tích bài toán quản lý tập trung cho **Hệ thống Quản lý Tòa nhà (BMS)** đa chi nhánh.
- Thiết kế **Kiến trúc hệ thống 5 tầng** cho Enterprise IoT Cloud Dashboard.
- Lựa chọn và đánh giá các **Dịch vụ AWS** (EC2, RDS, CloudWatch) phù hợp cho hạ tầng Backend.
- Định nghĩa **Mô hình dữ liệu** (Telemetry, Command) và chuẩn hóa cấu trúc REST API.
- Khởi tạo repository dự án, thiết lập quản lý phiên bản và phân chia công việc trong nhóm.

### Công việc thực hiện trong tuần này:

| Day | Task                                                                                                                                                                                                                                                                                                                                                                                                                                     | Start Date | Completion Date |
| :-- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------- | :-------------- |
| 1   | **Phân tích yêu cầu bài toán (Business Requirement)** <br> - **Xác định vấn đề:** Phân tích nhu cầu giám sát và điều khiển tập trung các tòa nhà thông minh ở nhiều vị trí địa lý khác nhau (Hà Nội, Đà Nẵng, TP.HCM). <br> - **Phạm vi hệ thống:** Chốt danh sách cảm biến (nhiệt độ, độ ẩm, ánh sáng) và thiết bị cần điều khiển (quạt, đèn, rèm).                                                                                     | 08/09/2025 | 08/09/2025      |
| 2   | **Thiết kế kiến trúc hệ thống (System Architecture)** <br> - **Mô hình 5 tầng:** Vẽ sơ đồ kiến trúc tổng thể từ Thiết bị IoT (Simulator/YOLO Uno) ➔ HTTP REST API ➔ FastAPI (EC2) ➔ PostgreSQL ➔ React Dashboard. <br> - **Luồng dữ liệu:** Phân tích luồng đẩy dữ liệu (Telemetry) từ thiết bị lên Cloud và luồng điều khiển (Command) từ Cloud xuống thiết bị.                                                                     | 09/09/2025 | 09/09/2025      |
| 3   | **Quy hoạch hạ tầng Cloud** <br> - **Lựa chọn dịch vụ:** Đánh giá việc sử dụng AWS EC2 để chạy FastAPI Backend và AWS RDS (PostgreSQL) để lưu trữ dữ liệu chuỗi thời gian của cảm biến. <br> - **Chiến lược giám sát:** Lên kế hoạch tích hợp AWS CloudWatch để ghi log hoạt động của Backend và theo dõi tình trạng hệ thống.                                                                                                         | 10/09/2025 | 10/09/2025      |
| 4   | **Thiết kế Database & API Contract** <br> - **Mô hình Database:** Phác thảo cấu trúc bảng PostgreSQL để lưu lịch sử đo đạc của thiết bị và trạng thái các lệnh điều khiển. <br> - **Đặc tả API:** Lập tài liệu cho các REST API core (VD: `POST /telemetry`, `POST /command`), quy định rõ cấu trúc JSON cho Request/Response.                                                                                                         | 11/09/2025 | 11/09/2025      |
| 5   | **Khởi tạo dự án & Môi trường làm việc** <br> - **Thiết lập Git:** Khởi tạo repository trên GitHub, thiết lập các nhánh làm việc (branching rules) và viết file `README.md` nền tảng. <br> - **Phân công nhiệm vụ:** Chốt rõ vai trò của 4 thành viên (Cloud, Backend, Frontend, IoT) để đảm bảo tiến độ công việc không bị chồng chéo.                                                                                                | 12/09/2025 | 12/09/2025      |

### Thành tựu Tuần 1:

- **Hoàn thiện Kiến trúc Hệ thống**
  - Thiết lập thành công bản thiết kế kiến trúc IoT 5 tầng rõ ràng, dễ mở rộng.
  - Tách bạch hoàn toàn phần cứng và Backend, đảm bảo tương lai có thể thay thế Python Simulator bằng phần cứng thật (YOLO Uno/ESP32) mà không làm vỡ hệ thống.

- **Chuẩn hóa Thông số Kỹ thuật**
  - Hoàn thành thiết kế lược đồ Cơ sở dữ liệu (Database Schema) cho PostgreSQL.
  - Chốt cấu trúc dữ liệu JSON giao tiếp giữa thiết bị và Cloud.

- **Sẵn sàng triển khai**
  - Repository đã được cấu hình chuẩn chỉ.
  - Các thành viên đã nắm rõ vai trò kỹ thuật và lộ trình phát triển của các tuần tiếp theo.

---

👉 **Kết quả:** Sau Tuần 1, nền tảng lý thuyết và bản vẽ kiến trúc của hệ thống Enterprise IoT Dashboard đã hoàn toàn vững chắc. Nhóm đã sẵn sàng bắt tay vào viết code cho Python Simulator và Backend API trong Tuần 2.