---
title: "Cấu hình VPC & Networking"
date: "2026-06-15"
weight: 2
chapter: false
pre: " <b> 5.2.2 </b> "
---

#### Tổng quan

Khởi tạo Đám mây riêng ảo (VPC) để làm nền tảng mạng bảo mật cho hệ thống Enterprise IoT Cloud Dashboard. Đây đóng vai trò là môi trường cô lập, nơi cơ sở dữ liệu PostgreSQL RDS và EC2 backend sẽ hoạt động an toàn. Bạn sẽ thiết kế VPC, các mạng con public/private, Internet Gateway và Route Tables để thiết lập kiến trúc cốt lõi.

#### Chuẩn bị Mạng lưới

Chúng ta sẽ cấu hình VPC và cung cấp các tài nguyên điện toán nền tảng cho backend.

**Bước 1. Thiết kế VPC và Mạng**

- Truy cập dịch vụ **VPC** từ thanh tìm kiếm.
- **AWS Region:** Chọn `United States (N. Virginia us-east-1)`.

![Truy cập VPC](/images/5-Workshop/5.2-Prerequisite/04_VPC.jpg)

- Click **Create VPC**.

![Create VPC](/images/5-Workshop/5.2-Prerequisite/05_VPC_Create.jpg)

- Cấu hình thông tin Mạng:
    - Thiết kế VPC với các subnets public và private.
    - Cấu hình Internet Gateway để cho phép lưu lượng truy cập từ bên ngoài.
    - Thiết lập Route Tables để quản lý luồng dữ liệu mạng.

![Configure VPC](/images/5-Workshop/5.2-Prerequisite/06_Configure_VPC.jpg)

- Kéo xuống cuối trang, Click **Create VPC**.

![Ảnh minh họa giao diện tạo VPC](/images/5-Workshop/5.2-Prerequisite/07_Finished_Create_VPC.jpg)

- Kiểm tra tạo VPC thành công.

![Create VPC Successful](/images/5-Workshop/5.2-Prerequisite/08_Create_VPC_Successful.jpg)

**Bước 2. Cấp phát EC2 & Security Groups**

- Khi mạng đã sẵn sàng, chúng ta cần cấp phát lớp điện toán cho FastAPI backend.

- Truy cập dịch vụ **EC2**.
![Click_EC2](/images/5-Workshop/5.2-Prerequisite/09_Click_EC2.jpg)

- Click **Launch instances**.
![Launch_Instance](/images/5-Workshop/5.2-Prerequisite/10_Launch_Instance.jpg)

- Tại giao diện khởi tạo:
    - Chọn Ubuntu EC2 instance để chạy FastAPI backend.
    - Gắn Elastic IP để đảm bảo một điểm cuối công khai tĩnh (static public endpoint).
    - Cấu hình Security Groups để đặc biệt cho phép truy cập HTTP và SSH.
- Kéo xuống cuối trang, Click **Launch instance**.

![Configure EC2](/images/5-Workshop/5.2-Prerequisite/11_Configure_EC2.jpg)

- Khi thấy thông báo thành công, Click **Close**.

![Launch successfully](/images/5-Workshop/5.2-Prerequisite/12_Successfully.jpg)