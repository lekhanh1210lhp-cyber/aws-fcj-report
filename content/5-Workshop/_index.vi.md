---
title: "Hướng dẫn Dự án"
date: "2026-06-15"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Hướng dẫn Triển khai Dự án Enterprise IoT Cloud Dashboard

#### Tổng quan

**Enterprise IoT Cloud Dashboard** là một giải pháp quản lý tòa nhà thông minh (BMS) tập trung được xây dựng trên nền tảng AWS. Hệ thống tích hợp giao diện React frontend, FastAPI backend chạy trên AWS EC2, cơ sở dữ liệu PostgreSQL RDS, giám sát bằng CloudWatch và Python Simulator đóng vai trò các thiết bị IoT.

> Giải pháp được thiết kế xung quanh 5 lớp cốt lõi nhằm đảm bảo quá trình thu thập dữ liệu viễn trắc thông suốt, thực thi lệnh điều khiển hai chiều đáng tin cậy và trực quan hóa thời gian thực trên nhiều địa điểm tòa nhà (Hà Nội, Đà Nẵng và TP. Hồ Chí Minh).

Trong hướng dẫn này, chúng ta sẽ xem xét toàn bộ vòng đời của dự án—từ việc thiết lập hạ tầng đám mây ban đầu và cấp phát cơ sở dữ liệu, triển khai REST API, mô phỏng thiết bị đa tòa nhà, giao tiếp hai chiều, phát triển bảng điều khiển frontend, tích hợp hệ thống, kiểm thử bảo mật chịu tải cho đến hoàn thiện tài liệu bàn giao dự án.

Chúng ta sẽ sử dụng các thành phần chính để thiết lập một giải pháp đám mây IoT hoàn chỉnh:

- **Frontend (React & TailwindCSS)** - Đóng vai trò là giao diện người dùng để giám sát viễn trắc thời gian thực, trực quan hóa dữ liệu lịch sử (qua Chart.js/Recharts) và điều khiển thiết bị từ xa (Quạt, Đèn, Rèm).
- **Backend & Database (FastAPI trên EC2 & PostgreSQL RDS)** - Xử lý xác thực dữ liệu thu thập qua Pydantic, quản lý lược đồ dữ liệu quan hệ bằng SQLAlchemy/Alembic và xếp hàng các lệnh điều khiển từ xa một cách an toàn.
- **IoT Simulation & Monitoring (Python Simulator & CloudWatch)** - Mô phỏng lưu lượng đa tòa nhà đồng thời bằng luồng (threading), xử lý các ngoại lệ mạng và giám sát tỷ lệ lỗi API cùng dấu vết kiểm toán.

#### Kết quả đạt được

Khi hoàn thành hướng dẫn dự án này, bạn sẽ có một giải pháp Enterprise IoT Cloud hoạt động đầy đủ với các tính năng sau:

- Quản lý tòa nhà thông minh tập trung trên nhiều vùng miền.
- Thu thập dữ liệu viễn trắc thời gian thực và bảng điều khiển phân tích lịch sử.
- Giao tiếp hai chiều hỗ trợ thực thi lệnh từ xa và polling thiết bị.
- Hạ tầng đám mây được bảo mật với tính năng giới hạn tốc độ API, cô lập security group và kiểm thử độ ổn định dưới tải nặng.

#### Nội dung

1. [Kiến trúc Hệ thống & Nền tảng Hạ tầng Đám mây](5.1-Workshop-overview/)
2. [Thiết kế Cơ sở dữ liệu & Nền tảng Backend](5.2-Prerequiste/)
3. [Triển khai REST API (Thu thập Dữ liệu)](5.3-Knowledge-Base/)
4. [Mô phỏng Thiết bị IoT](5.4-Test-Chatbot/)
5. [Thực thi Lệnh & Giao tiếp Hai chiều](5.5-Client-Integration/)
6. [Giao diện Frontend Dashboard & Bảng điều khiển](5.6-Cleanup/)
7. [Tích hợp Toàn trình, Bảo mật & Bàn giao](5.7-Cleanup/)