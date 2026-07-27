---
title: "Worklog Tuần 3"
date: "2026-06-29"
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

- Xây dựng và kiểm thử các API endpoint để thu thập dữ liệu viễn trắc (telemetry ingestion).
- Thiết kế và triển khai các RESTful API endpoint bằng FastAPI.
- Thiết lập cơ chế xác thực dữ liệu chặt chẽ bằng Pydantic để từ chối các payload không hợp lệ.
- Phát triển tính năng truy xuất dữ liệu lịch sử có hỗ trợ phân trang (pagination).
- Đảm bảo độ tin cậy của API thông qua kiểm thử Postman toàn diện và giám sát bằng AWS CloudWatch.

### Công việc thực hiện trong tuần này:

| Day | Task | Start Date | Completion Date | Reference Material |
| :-- | :--- | :--------- | :-------------- | :----------------- |
| 1   | **POST /telemetry:** Phát triển endpoint để nhận dữ liệu nhiệt độ, độ ẩm, ánh sáng và trạng thái thiết bị. | 29/06/2026 | 30/06/2026 | Tài liệu API Backend |
| 2   | **Data Validation:** Triển khai các validator bằng Pydantic để từ chối các payload dữ liệu không đúng định dạng. | 01/07/2026 | 01/07/2026 | Tài liệu FastAPI & Pydantic |
| 3   | **GET /telemetry:** Phát triển các endpoint để lấy trạng thái mới nhất và dữ liệu lịch sử có phân trang. | 02/07/2026 | 03/07/2026 | Tài liệu API Backend |
| 4   | **Postman Testing:** Viết và thực thi các bộ kiểm thử Postman toàn diện cho API. | 04/07/2026 | 04/07/2026 | Bộ kiểm thử Postman |
| 5   | **CloudWatch Logs:** Cloud Engineer tích hợp AWS CloudWatch để giám sát tỷ lệ lỗi của API. | 05/07/2026 | 05/07/2026 | Tài liệu AWS CloudWatch |

### Thành tựu Tuần 3:

- **Phát triển API:** Backend nhận và xác thực thành công dữ liệu JSON.
- **Tích hợp Database:** Dữ liệu sau khi xác thực được lưu trữ chính xác vào cơ sở dữ liệu PostgreSQL.
- **Xác thực dữ liệu:** Triển khai thành công Pydantic schema để loại bỏ các payload IoT lỗi.
- **Kiểm thử & Giám sát:** Hoàn thành bộ kiểm thử API trên Postman và tích hợp AWS CloudWatch để theo dõi tỷ lệ lỗi.