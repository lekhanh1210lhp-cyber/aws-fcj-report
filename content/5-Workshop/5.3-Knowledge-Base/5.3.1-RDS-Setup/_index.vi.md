---
title: "Thiết lập AWS RDS"
date: "2026-06-15"
weight: 1
chapter: false
pre: " <b> 5.3.1 </b> "
---

#### Mục tiêu

Chúng ta sẽ sử dụng AWS Management Console để khởi tạo một PostgreSQL RDS instance. Quá trình này sẽ triển khai cơ sở dữ liệu trong một mạng con dùng riêng (private subnet) và cấu hình các quy tắc đầu vào (inbound rules) để chỉ chấp nhận lưu lượng truy cập từ EC2 nhằm đảm bảo tính bảo mật.

#### Các Bước Thực hiện

1.  Đăng nhập vào **AWS Management Console** và truy cập dịch vụ **RDS**.
2.  Trong menu bên trái, chọn **Databases**.

![Click_Databases](/images/5-Workshop/5.3-RDS-Setup/01_Click_Databases.jpg)

3.  Nhấp vào nút **Create database** ở góc trên bên phải của màn hình.

![Create Database](/images/5-Workshop/5.3-RDS-Setup/02_Create_Database.jpg)

**Bước 1: Chọn phương thức tạo và Engine**

Trên màn hình cấu hình đầu tiên:

1. **Choose a database creation method:** Chọn `Standard create`.
2. **Engine options:** Chọn `PostgreSQL`.
3. **Templates:** Chọn `Free tier` hoặc `Dev/Test` tùy thuộc vào cấu hình ngân sách của bạn.

![Configure Engine](/images/5-Workshop/5.3-RDS-Setup/03_Configure_Engine.jpg)

**Bước 2: Thiết lập thông tin chung (Settings)**

Cấu hình các thông tin cơ sở dữ liệu cốt lõi:

1.  **DB instance identifier:** Nhập `iot-dashboard-db`
2.  **Master username:** Nhập `postgres` (hoặc tên quản trị viên bạn muốn).
3.  **Master password:** Nhập mật khẩu mạnh và xác nhận lại. Lưu trữ an toàn mật khẩu này để cấu hình kết nối cho FastAPI backend sau này.

![Configure Settings](/images/5-Workshop/5.3-RDS-Setup/04_Configure_Settings.jpg)

**Bước 3: Cấu hình Kết nối (Connectivity)**

Đây là bước quan trọng nhất để bảo mật tầng cơ sở dữ liệu:

1.  **Virtual private cloud (VPC):** Chọn VPC tùy chỉnh mà bạn đã tạo ở các bước trước.
2.  **Public access:** Chọn `No` (Điều này đảm bảo cơ sở dữ liệu được triển khai trong private subnet).
3.  **VPC security group (firewall):** Chọn `Create new` (ví dụ: `rds-ec2-sg`) hoặc chọn group đã có.
4.  *Lưu ý:* Bạn phải cấu hình inbound rules của Security Group này để cho phép lưu lượng PostgreSQL (Port 5432) **chỉ từ EC2 instance** đang chạy FastAPI backend.

![Configure Connectivity](/images/5-Workshop/5.3-RDS-Setup/05_Configure_Connectivity.jpg)

**Bước 4: Kiểm tra và Tạo Database**

1.  Kiểm tra tất cả thông tin cấu hình trên trang tóm tắt.
2.  Đảm bảo cấu hình VPC, Subnet và Public Access (No) đã chính xác.
3.  Cuộn xuống cuối trang và nhấp vào nút **Create database**.

![Step 4](/images/5-Workshop/5.3-RDS-Setup/06_Step_4.jpg)

**Bước 5: Chờ Khởi tạo**

Sau khi nhấp Create, hệ thống sẽ bắt đầu quá trình cấp phát RDS instance.

- **Thời gian chờ:** Khoảng **5 - 10 phút**.
- **Lưu ý:** Bạn có thể chuyển sang trang khác, nhưng hãy đợi quá trình hoàn tất trước khi kết nối backend.
- **Thành công:** Khi trạng thái cơ sở dữ liệu chuyển sang **"Available"**, bạn đã hoàn thành bước này và sẵn sàng khởi tạo các schemas cho FastAPI backend.

![Step 5](/images/5-Workshop/5.3-RDS-Setup/07_Step_5.jpg)