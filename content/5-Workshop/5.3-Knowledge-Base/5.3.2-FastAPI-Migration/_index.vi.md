---
title: "Khởi tạo FastAPI & Database Migration"
date: "2026-06-15"
weight: 2
chapter: false
pre: " <b> 5.3.2 </b> "
---

#### Mục tiêu

Trước khi backend có thể nhận dữ liệu viễn trắc, chúng ta phải khởi tạo dự án FastAPI và chuẩn bị cấu trúc cơ sở dữ liệu. Chúng ta sẽ cấu hình SQLAlchemy và Pydantic schemas, thiết lập Alembic và thực thi bản migration đầu tiên lên PostgreSQL RDS instance.

#### Các Bước Thực hiện

**Bước 1: Khởi tạo FastAPI & Schemas**

Backend Engineer sẽ bắt đầu thiết lập cấu trúc lõi cho backend.

1.  Khởi tạo môi trường dự án FastAPI trên máy local hoặc trực tiếp trên EC2 instance.
    ![FastAPI Init](/images/5-Workshop/5.3-FastAPI-Migration/13_FastAPI_Init.jpg)

2.  Cấu hình Pydantic schemas. Điều này đảm bảo backend có thể xác thực các payload dữ liệu JSON đầu vào và từ chối các dữ liệu viễn trắc bị lỗi định dạng sau này.
    ![Pydantic Schemas](/images/5-Workshop/5.3-FastAPI-Migration/14_Pydantic.jpg)

3.  Thiết lập SQLAlchemy models để định nghĩa các bảng quan hệ. Bạn sẽ cần thiết kế các bảng cho Buildings (Tòa nhà), Telemetry History (Lịch sử viễn trắc) và Commands (Lệnh điều khiển).
    ![SQLAlchemy Models](/images/5-Workshop/5.3-FastAPI-Migration/15_SQLAlchemy.jpg)

**Bước 2: Cấu hình Alembic**

Bây giờ chúng ta sẽ chuẩn bị môi trường migration để triển khai lược đồ cơ sở dữ liệu lên AWS RDS.

1.  Thiết lập Alembic cho việc quản lý cấu trúc dữ liệu (schema migrations).
2.  Cấu hình file `alembic.ini` và `env.py` để trỏ chuỗi kết nối cơ sở dữ liệu đến PostgreSQL RDS instance vừa được triển khai.
3.  Tạo (Generate) kịch bản migration ban đầu dựa trên các SQLAlchemy models mà bạn đã định nghĩa.

![Alembic Setup](/images/5-Workshop/5.3-FastAPI-Migration/16_Alembic_Setup.jpg)

**Bước 3: Thực thi Migration lên RDS**

Cuối cùng, chúng ta áp dụng cấu trúc cơ sở dữ liệu lên RDS instance thực tế để hoàn thiện nền tảng backend.

1.  Thực thi lệnh migration đầu tiên (ví dụ: `alembic upgrade head`) để đẩy các thay đổi schema lên RDS.
2.  Kết nối với cơ sở dữ liệu PostgreSQL của bạn (sử dụng các công cụ như pgAdmin hoặc DBeaver) để xác minh việc triển khai.
3.  **Kết quả:**
    - Cấu trúc cơ sở dữ liệu (schema) đã được triển khai thành công.
    - Các bảng quan hệ (Buildings, Telemetry History, Commands) đã được tạo chính xác và hoạt động bình thường.

![Migration Success](/images/5-Workshop/5.3-FastAPI-Migration/17_Migration_Success.jpg)

**Chúc mừng!** Bạn đã khởi tạo thành công cấu trúc backend FastAPI và triển khai cấu trúc cơ sở dữ liệu. Máy chủ backend hiện đã sẵn sàng cho giai đoạn tiếp theo: Triển khai REST API để thu thập dữ liệu.