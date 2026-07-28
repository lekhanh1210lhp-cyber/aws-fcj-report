---
title: "Thiết kế Cơ sở dữ liệu & Nền tảng Backend"
date: "2026-06-15"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Mục tiêu

Sau khi hoàn tất việc chuẩn bị môi trường AWS, bước tiếp theo là xây dựng lớp dữ liệu và nền tảng backend cho dashboard IoT. Phần này tập trung vào việc thiết lập cơ sở dữ liệu và API backbone để sau này có thể nhận dữ liệu viễn trắc từ các thiết bị mô phỏng và cung cấp cho frontend.

Chúng ta sẽ hoàn thành ba mục tiêu kỹ thuật chính:

1. **Cấp phát tầng dữ liệu** bằng Amazon RDS cho PostgreSQL trong subnet riêng.
2. **Khởi tạo backend FastAPI** và định nghĩa các model đầu vào, đầu ra cho hệ thống.
3. **Áp dụng migration database** để lưu trữ dữ liệu về tòa nhà, lịch sử viễn trắc và lệnh điều khiển.

#### Thành phần chính

Trong phần này, chúng ta sẽ làm việc với các dịch vụ và công cụ sau:

- **AWS RDS (PostgreSQL):** Dịch vụ cơ sở dữ liệu quan hệ được quản lý, dùng để lưu trữ dữ liệu IoT có cấu trúc như thông tin tòa nhà, lịch sử telemetry và nhật ký lệnh.
- **FastAPI (Python):** Framework backend hiện đại dùng để xử lý yêu cầu API, xác thực payload và kết nối với database.
- **SQLAlchemy và Alembic:** Công cụ để định nghĩa mô hình dữ liệu và triển khai thay đổi schema một cách an toàn.

#### Quy trình thực hiện

1. Tạo instance PostgreSQL trên AWS RDS và đảm bảo chỉ có backend EC2 mới có thể kết nối.
2. Cấu hình FastAPI để kết nối tới database bằng biến môi trường và thông tin xác thực bảo mật.
3. Tạo schema ban đầu và áp dụng migration đầu tiên.
4. Kiểm tra các bảng dữ liệu đã sẵn sàng để nhận dữ liệu viễn trắc.

<!-- Chèn ảnh: giao diện AWS RDS khi tạo instance PostgreSQL -->
> Chỗ dành cho ảnh: màn hình AWS RDS và cấu hình security group.

#### Kết quả mong đợi

Sau khi hoàn thành phần này, hệ thống sẽ có tầng dữ liệu và backend nền tảng ổn định để phục vụ cho việc kiểm thử API và tích hợp frontend.