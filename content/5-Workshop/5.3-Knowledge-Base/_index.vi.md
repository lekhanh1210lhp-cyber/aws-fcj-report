---
title: "Thiết kế Cơ sở dữ liệu & Nền tảng Backend"
date: "2026-06-15"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Mục tiêu

Sau khi hoàn thành việc chuẩn bị nền tảng hạ tầng đám mây, bước tiếp theo là thiết lập các thành phần cơ sở dữ liệu và backend cốt lõi của kiến trúc IoT. Trong phần này, chúng ta sẽ cung cấp PostgreSQL trên AWS RDS và khởi tạo cấu trúc backend FastAPI.

Chúng ta sẽ thực hiện 3 mục tiêu kỹ thuật chính:

1.  **Khởi tạo Cơ sở dữ liệu:** Triển khai một instance PostgreSQL RDS trong mạng con dùng riêng (private subnet) và cấu hình các quy tắc đầu vào (inbound rules) chỉ cho phép kết nối từ EC2.
2.  **Khởi tạo FastAPI:** Khởi tạo dự án FastAPI từ đầu, tiến hành cấu hình SQLAlchemy và các Pydantic schemas.
3.  **Database Migration:** Thiết lập Alembic cho việc di chuyển (migration) cấu trúc dữ liệu và thực thi bản migration đầu tiên lên RDS để tạo các bảng quan hệ cho Buildings, Telemetry History, và Commands.

#### Các Thành phần Chính

Trong quá trình cấu hình này, chúng ta sẽ tương tác và kết nối các dịch vụ cũng như công cụ sau:

- **AWS RDS (PostgreSQL):** Dịch vụ cơ sở dữ liệu quan hệ được quản lý, dùng để lưu trữ an toàn các dữ liệu có cấu trúc bao gồm Buildings, Telemetry History, và Commands.
- **FastAPI (Python):** Framework backend hiện đại đang hoạt động tích cực trên EC2 instance, chịu trách nhiệm xử lý logic API và xác thực dữ liệu.
- **Alembic & SQLAlchemy:** Bộ công cụ cơ sở dữ liệu và công cụ di chuyển (migration) được sử dụng để thiết lập các lược đồ quan hệ và thực thi triển khai trực tiếp lên RDS instance.

#### Các Bước Thực hiện

1. [Thiết lập AWS RDS](5.3.1-RDS-Setup/)
2. [Khởi tạo FastAPI & Database Migration](5.3.2-FastAPI-Migration/)