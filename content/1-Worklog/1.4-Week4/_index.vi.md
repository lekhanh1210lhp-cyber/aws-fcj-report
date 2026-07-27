---
title: "Worklog Tuần 4"
date: "2026-07-06"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

- Phát triển **Python Simulator** để đóng vai trò như các thiết bị edge YOLO Uno/ESP32 cho hệ thống Enterprise IoT Cloud Dashboard.
- Thiết lập cơ chế đáng tin cậy để tạo và truyền dữ liệu viễn trắc (telemetry) mô phỏng trực tiếp đến backend trên EC2.
- Đảm bảo độ bền bỉ của hệ thống thông qua xử lý đa luồng (multi-threading), bắt lỗi và kiểm thử tích hợp toàn trình.

### Công việc thực hiện trong tuần này:

| Day | Task | Start Date | Completion Date | Reference Material |
| :-- | :--- | :--------- | :-------------- | :----------------- |
| 1   | **Thiết kế Simulator** <br> - **Tạo dữ liệu:** Kỹ sư IoT (IoT Engineer) tạo các script Python để sinh ra dữ liệu cảm biến ngẫu nhiên, sát với thực tế. | 06/07/2026 | 07/07/2026 | Tài liệu Python `random` |
| 2   | **Thiết lập HTTP Client** <br> - **Giao tiếp API:** Triển khai thư viện `requests` để gửi dữ liệu JSON (POST) đến public IP của FastAPI backend. | 08/07/2026 | 08/07/2026 | Tài liệu Python `requests` |
| 3   | **Mô phỏng đa tòa nhà (Multi-Building)** <br> - **Mở rộng quy mô:** Sử dụng threading (đa luồng) để script có thể mô phỏng đồng thời lưu lượng dữ liệu từ các tòa nhà ở HN, ĐN và HCM. | 09/07/2026 | 10/07/2026 | Tài liệu Python `threading` |
| 4   | **Xử lý lỗi (Error Handling)** <br> - **Khả năng chịu lỗi mạng:** Thêm logic thử lại (retry) và bắt ngoại lệ (exception handling) cho các trường hợp rớt kết nối mạng. | 11/07/2026 | 11/07/2026 | Tài liệu Kiến trúc Hệ thống |
| 5   | **Kiểm thử Tích hợp (Integration Test)** <br> - **Xác minh toàn trình:** Kiểm tra và xác nhận dữ liệu từ tất cả các tòa nhà mô phỏng đã xuất hiện thành công trong cơ sở dữ liệu PostgreSQL. | 12/07/2026 | 12/07/2026 | Tài liệu PostgreSQL |

### Thành tựu Tuần 4:

- **Mô phỏng IoT thành công:** Các script Python hoạt động ổn định trong việc tạo và gửi dữ liệu viễn trắc về backend EC2.
- **Khả năng mở rộng:** Đã mô phỏng thành công các luồng dữ liệu đồng thời từ nhiều vị trí tòa nhà (HN, ĐN, HCM) bằng kỹ thuật đa luồng.
- **Độ tin cậy của hệ thống:** Cải thiện tính ổn định của bộ mô phỏng bằng cách triển khai cơ chế bắt lỗi và thử lại khi rớt mạng.
- **Luồng dữ liệu được xác minh:** Xác nhận tích hợp thành công, dữ liệu cảm biến mô phỏng được lưu trữ chính xác vào cơ sở dữ liệu PostgreSQL.