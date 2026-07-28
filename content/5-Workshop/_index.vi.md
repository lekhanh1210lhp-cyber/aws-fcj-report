---
title: "Workshop AWS IoT Dashboard"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# AWS IoT Monitoring and Control Dashboard

Workshop này xây dựng hệ thống giám sát môi trường và điều khiển thiết bị hoàn chỉnh cho `room_01`. YOLO UNO đọc nhiệt độ, độ ẩm và ánh sáng; FastAPI trên Amazon EC2 lưu telemetry vào Amazon RDS for PostgreSQL; React dashboard hiển thị dữ liệu hiện tại và lịch sử; Amazon CloudWatch cung cấp log, metric và alarm.

Luồng điều khiển có thể kiểm chứng rõ ràng: dashboard tạo command ở trạng thái `Pending`, thiết bị polling và thực thi lệnh, sau đó gửi xác nhận (ACK) để chuyển trạng thái sang `Executed`.

## Kết quả học tập

Sau khi hoàn thành workshop, bạn có thể:

- triển khai FastAPI trên EC2 và kết nối an toàn tới RDS;
- gửi telemetry từ YOLO UNO và lưu dữ liệu trong PostgreSQL;
- điều khiển quạt, đèn và rèm từ React dashboard;
- kiểm thử toàn bộ vòng đời command và ACK;
- thu thập bằng chứng vận hành bằng CloudWatch; và
- ước tính chi phí, áp dụng bảo mật cơ bản và xóa tài nguyên tính phí.

## Nội dung workshop

1. [Tổng quan Workshop](5.1-Workshop-Overview/)
2. [Điều kiện tiên quyết](5.2-Prerequisites/)
3. [Kiến trúc và Thiết kế Dịch vụ](5.3-Architecture-and-Service-Design/)
4. [Thiết lập Hạ tầng AWS](5.4-AWS-Infrastructure-Setup/)
5. [Triển khai Backend và Tích hợp Cơ sở dữ liệu](5.5-Backend-and-Database/)
6. [Tích hợp Phần cứng](5.6-Hardware-Integration/)
7. [Tích hợp Frontend](5.7-Frontend-Integration/)
8. [Kiểm thử và Xác thực End-to-End](5.8-End-to-End-Testing/)
9. [Giám sát bằng CloudWatch](5.9-CloudWatch-Monitoring/)
10. [Chi phí, Bảo mật và Dọn dẹp](5.10-Cost-Security-Cleanup/)
11. [Kết quả, Thách thức và Hướng phát triển](5.11-Results-Challenges-Future/)
12. [Bàn giao Dự án](5.12-Project-Handover/)

> Sử dụng nhất quán Region Singapore (`ap-southeast-1`). Thay mọi giá trị `<PLACEHOLDER>` bằng thông tin của bạn và không công khai mật khẩu, token, thông tin Wi-Fi hoặc private key.
