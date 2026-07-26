---
title: "Worklog Tuần 2"
date: "2026-06-22"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

- Cấp phát (Provision) cơ sở dữ liệu PostgreSQL trên AWS RDS.
- Khởi tạo cấu trúc backend FastAPI.

### Công việc thực hiện trong tuần này:

| Day | Task | Start Date | Completion Date |
| :-- | :--- | :--------- | :-------------- |
| 1 | **Database Schema:** Thiết kế các bảng quan hệ cho Buildings, Telemetry History, và Commands. | 22/06/2026 | 22/06/2026 |
| 2 | **Thiết lập AWS RDS:** Triển khai PostgreSQL RDS instance trong private subnet, cấu hình inbound rules chỉ cho phép truy cập từ EC2. | 23/06/2026 | 24/06/2026 |
| 3 | **Khởi tạo FastAPI:** Kỹ sư Backend khởi tạo dự án FastAPI, cấu hình SQLAlchemy và Pydantic schemas. | 25/06/2026 | 26/06/2026 |
| 4 | **Database Migration:** Thiết lập Alembic để thực hiện migrate schema và chạy bản migration đầu tiên lên RDS. | 25/06/2026 | 27/06/2026 |
| 5 | **Bản nháp CI/CD:** Viết nháp các script triển khai để tự động pull code và khởi động lại các systemctl services trên EC2. | 28/06/2026 | 28/06/2026 |

### Thành tựu Tuần 2:

- Database schema đã được triển khai thành công.
- Backend server đang hoạt động ổn định trên EC2.

---

👉 **Kết quả:** Sau Tuần 2, cơ sở dữ liệu quan hệ và nền tảng REST API đã được thiết lập thành công, sẵn sàng cho việc tiếp nhận dữ liệu (Data Ingestion) vào Tuần 3.