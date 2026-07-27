---
title: "Bản đề xuất"
date: "2026-06-15"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Enterprise IoT Cloud Dashboard

### Bạn có thể đọc toàn bộ proposal ở đây: <a href="/files/2-Proposal/IoT_Dashboard_Proposal.pdf" download>IoT Dashboard Proposal</a>

### 1. Tóm tắt điều hành

Dự án là một hệ thống Enterprise IoT Cloud Dashboard trên AWS, được thiết kế để quản lý tòa nhà thông minh tập trung (BMS). Hệ thống sử dụng React cho frontend, FastAPI trên AWS EC2 cho backend, PostgreSQL RDS làm cơ sở dữ liệu, CloudWatch để giám sát và Python Simulator cho các thiết bị IoT. Quá trình phát triển dự án bắt đầu từ ngày 15/06/2026.

### 2. Tuyên bố vấn đề

#### Vấn đề hiện tại & Giải pháp
Dự án giải quyết nhu cầu quản lý tòa nhà thông minh tập trung thông qua nền tảng đám mây. Giải pháp cốt lõi là hoàn thiện kiến trúc 5 lớp. Hệ thống bao gồm việc thu thập dữ liệu viễn trắc (nhận thông tin nhiệt độ, độ ẩm, ánh sáng và trạng thái thiết bị) và cho phép thực thi lệnh điều khiển từ xa (ví dụ: Bật/Tắt quạt, Mở rèm).

#### Lợi ích
Hệ thống thiết lập giao tiếp hai chiều hoàn chỉnh giữa Simulator và Cloud. Nó đảm bảo các thiết bị biên (mô phỏng phần cứng YOLO Uno hoặc ESP32) có thể tạo và gửi dữ liệu viễn trắc ổn định về backend, đồng thời nhận các lệnh đang chờ xử lý.

### 3. Kiến trúc giải pháp

Kiến trúc bao gồm 5 lớp, được phân chia vai trò nghiêm ngặt cho các thành viên trong nhóm.

#### Chi tiết Tech Stack:
| Thành phần | Công nghệ | Chi tiết |
| :--- | :--- | :--- |
| **Frontend** | **React** | Sử dụng Vite-React, TailwindCSS và routing. Tích hợp Axios để gọi API và Chart.js/Recharts để vẽ biểu đồ. |
| **Backend** | **FastAPI (Python)** | Lưu trữ trên Ubuntu EC2. Sử dụng SQLAlchemy và Pydantic schemas. |
| **Database** | **PostgreSQL** | Triển khai trên AWS RDS kèm Alembic để migrate cấu trúc dữ liệu. |
| **IoT / Edge** | **Python Simulator** | Mô phỏng thiết bị YOLO Uno/ESP32 sử dụng thư viện 'requests' và 'threading'. |
| **Monitoring** | **CloudWatch** | Tích hợp để giám sát tỷ lệ lỗi API và lưu vết kiểm toán cho các lệnh được thực thi. |

#### Luồng hoạt động trên AWS:
1. **Networking:** Thiết kế VPC với các subnet public/private, Internet Gateway và Route Tables.
2. **Compute:** Khởi tạo Ubuntu EC2 cho FastAPI backend, gắn Elastic IP và cấu hình Security Groups (HTTP/SSH).
3. **Database Security:** PostgreSQL RDS được triển khai trong private subnet với inbound rules chỉ cho phép truy cập từ EC2.
4. **Identity:** Thiết lập AWS IAM bao gồm users, groups, policies và bắt buộc MFA cho mọi tài khoản.

### 4. Triển khai kỹ thuật

#### Các giai đoạn triển khai
Dự án kéo dài chính xác 10 tuần:
1. **Nền tảng (Tuần 1-2):** Thiết lập môi trường AWS, VPC, EC2 và PostgreSQL RDS. Khởi tạo FastAPI và database schemas.
2. **Dữ liệu & IoT (Tuần 3-5):** Triển khai API thu thập dữ liệu, bộ mô phỏng Python và thiết lập giao tiếp hai chiều để thực thi lệnh.
3. **Frontend UI (Tuần 6-7):** Xây dựng Dashboard React, tích hợp API và trực quan hóa dữ liệu lịch sử.
4. **Tích hợp & Bảo mật (Tuần 8-10):** Kiểm thử End-to-End toàn trình, tối ưu độ trễ, củng cố bảo mật hạ tầng và hoàn thiện tài liệu LaTeX.

#### Yêu cầu kỹ thuật chi tiết:
- **CI/CD Draft:** Soạn thảo script deploy để kéo code và khởi động lại systemctl services trên EC2.
- **Xác thực dữ liệu:** Triển khai Pydantic validators để loại bỏ các payload dữ liệu lỗi.
- **Hàng đợi lệnh (Command Queue):** Logic trên PostgreSQL xếp hàng các lệnh chờ cho thiết bị, bộ mô phỏng sẽ định kỳ lấy dữ liệu (polling).

### 5. Lộ trình & Mốc triển khai (Sprints)

- **Tuần 1:** System Architecture & Cloud Infrastructure Foundation.
- **Tuần 2:** Database Design & Backend Foundation.
- **Tuần 3:** REST API Implementation (Data Ingestion).
- **Tuần 4:** IoT Device Simulation.
- **Tuần 5:** Command Execution & Two-Way Communication.
- **Tuần 6:** Frontend Development - Dashboard UI.
- **Tuần 7:** Frontend Development Control Panel & Analytics.
- **Tuần 8:** End-to-End System Integration.
- **Tuần 9:** System Security & Stress Testing.
- **Tuần 10:** Final Documentation & Pitch Preparation.

### 6. Ước tính ngân sách

Dự án chú trọng vào việc tối ưu hóa chi phí.
- **Định cỡ tài nguyên:** Đảm bảo tài nguyên được cấp phát phù hợp với các loại instance t2.micro hoặc t3.micro.
- **Giám sát:** Kiểm tra các cảnh báo thanh toán AWS (billing alerts).

### 7. Đánh giá rủi ro

- **Rủi ro Bảo mật:** Được xử lý thông qua Kiểm toán Bảo mật (Security Audit) để đảm bảo Security Groups cách ly DB khỏi public internet.
- **DDoS/Spam:** Được giảm thiểu bằng cách giới hạn tốc độ API (Rate Limiting) cơ bản trên FastAPI.
- **Quá tải hệ thống:** Được đánh giá qua Stress Testing, nơi simulator gửi dữ liệu tần số cao trong khi giám sát CPU của EC2.
- **Rớt mạng:** Được khắc phục bằng logic thử lại (retry logic) và xử lý ngoại lệ trong IoT Simulator khi rớt kết nối.

### 8. Kết quả kỳ vọng & Đội ngũ

#### Kết quả mong đợi của dự án
- **Vận hành gắn kết:** Hệ thống hoạt động thống nhất như một giải pháp Enterprise IoT Cloud duy nhất.
- **Sẵn sàng phần cứng:** Tài liệu được tinh chỉnh hướng dẫn rõ cách thay thế phần mềm mô phỏng bằng thiết bị YOLO Uno thực tế sau này.
- **Thành phẩm:** Dự án sẵn sàng để nộp báo cáo học thuật và có bài thuyết trình 30 giây chuẩn doanh nghiệp.

#### Đội ngũ thực hiện:

| Name | Role | Email | Trách nhiệm chính |
| :--- | :--- | :--- | :--- |
| **Phạm Lê Minh Khôi** | Cloud Developer (Leader) | khoi.pham2005ktmt@hcmut.edu.vn | Thiết lập IAM, VPC, EC2, RDS và CloudWatch. Đánh giá Security Groups. |
| **Ngô Minh Thuận** | Backend Developer (Member) | thuan.ngostudyhcmut@hcmut.edu.vn | Khởi tạo FastAPI, database schemas, migrate với Alembic và xử lý CORS. |
| **Thượng Đình Hưng** | Frontend Developer (Member) | hung.thuongpeanut2005@gmail.com | Khởi tạo dự án Vite-React, layout TailwindCSS, tích hợp API và hoàn thiện giao diện dashboard. |
| **Lê Bảo Khánh** | IoT Developer (Member) | lekhanh1210lhp@gmail.com | Tạo script mô phỏng Python, đa luồng (multithreading) và stress test tần số cao. |