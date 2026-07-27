---
title: "Giới thiệu"
date: "2026-06-15"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### Giới thiệu tổng quan

Trong dự án này, chúng ta sẽ tập trung xây dựng một hệ thống Enterprise IoT Cloud Dashboard trên AWS nhằm quản lý tòa nhà thông minh tập trung (BMS).

Mục tiêu chính là thiết lập một kiến trúc 5 lớp sử dụng React (Frontend), FastAPI trên AWS EC2 (Backend), PostgreSQL RDS (Database), CloudWatch (Monitoring) và Python Simulator (IoT Devices). Hệ thống bao gồm các bước:

1.  **Hạ tầng & Cơ sở dữ liệu:** Khởi tạo các tài nguyên AWS nền tảng (VPC, EC2) và triển khai cơ sở dữ liệu PostgreSQL RDS.
2.  **API & Mô phỏng IoT:** Xây dựng các API endpoint thu thập dữ liệu viễn trắc và phát triển kịch bản Python để mô phỏng thiết bị biên YOLO Uno/ESP32.
3.  **Frontend & Tích hợp:** Xây dựng bảng điều khiển React để trực quan hóa dữ liệu lịch sử và điều khiển thiết bị từ xa, thiết lập giao tiếp hai chiều hoàn chỉnh.

> 💡 **Điểm nổi bật:** Giải pháp này cho phép quản trị viên **xem các xu hướng dữ liệu và gửi lệnh trực tiếp từ giao diện (UI)**, đảm bảo luồng dữ liệu trơn tru từ Simulator đến Dashboard và ngược lại.

![overview](/images/5-Workshop/5.1-Workshop-overview/overview_diagram.png)

#### Các Bước Thực hiện

1. [Kiến trúc Hệ thống & Hạ tầng Đám mây](5.1.1-Architecture/)
2. [Thiết kế Cơ sở dữ liệu & Triển khai REST API](5.1.2-Backend/)