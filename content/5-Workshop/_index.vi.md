---
title: "Workshop"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

Workshop thực hành này xây dựng luồng end-to-end đã được tài liệu hóa cho một thiết bị vật lý `room_01`: YOLO UNO đọc DHT20 và cảm biến ánh sáng analog, FastAPI lưu telemetry và command trong Amazon RDS for PostgreSQL, React hiển thị dữ liệu và tạo command, còn thiết bị gửi ACK sau khi thực thi thao tác quạt, đèn và rèm.

## Mục tiêu và kết quả cuối

Sau Workshop, bạn có thể:

- triển khai backend FastAPI trên Amazon EC2 với EBS root volume;
- kết nối riêng EC2 tới Amazon RDS for PostgreSQL trong DB Subnet Group;
- tích hợp telemetry, polling command, thực thi và ACK trên YOLO UNO;
- kết nối dashboard React + Vite + TypeScript chạy local;
- xác minh command chuyển từ `Pending` sang `Executed`; và
- thu thập bằng chứng vận hành EC2, RDS và backend trên CloudWatch.

Kết quả mong đợi là một prototype có thể tái tạo cho `room_01`, không phải hệ thống BMS enterprise. Dự kiến cần **8-12 giờ tập trung** sau khi đã có source ứng dụng và tài khoản AWS.

## Bản đồ Workshop

1. [5.1 Tổng quan Workshop](5.1-Workshop-overview/)
2. [5.2 Điều kiện tiên quyết](5.2-Prerequisites/)
3. [5.3 Kiến trúc và thiết kế dịch vụ](5.3-Architecture-and-Service-Design/)
4. [5.4 Thiết lập hạ tầng AWS](5.4-AWS-Infrastructure-Setup/)
5. [5.5 Triển khai backend và tích hợp cơ sở dữ liệu](5.5-Backend-and-Database/)
6. [5.6 Tích hợp phần cứng](5.6-Hardware-Integration/)
7. [5.7 Tích hợp frontend](5.7-Frontend-Integration/)
8. [5.8 Kiểm thử và xác minh end-to-end](5.8-End-to-End-Testing/)
9. [5.9 Giám sát với CloudWatch](5.9-CloudWatch-Monitoring/)
10. [5.10 Chi phí, bảo mật và dọn dẹp](5.10-Cost-Security-Cleanup/)
11. [5.11 Kết quả, thách thức và cải tiến tương lai](5.11-Results-Challenges-Future/)
12. [5.12 Bàn giao dự án](5.12-Project-Handover/)

## Kiến trúc

![Kiến trúc AWS IoT Monitoring and Control Dashboard](/images/5-Workshop/5.3-architecture/aws-iot-dashboard-architecture.png)

*Hình 5-1. Sơ đồ từ source repository đặt người dùng dashboard, React frontend local và YOLO UNO bên ngoài AWS; EC2 và RDS bên trong VPC; IAM và CloudWatch là dịch vụ cấp tài khoản/khu vực nằm ngoài ranh giới VPC.*

Các dịch vụ AWS đã dùng gồm **Amazon EC2, Amazon EBS, Amazon RDS for PostgreSQL, Amazon VPC, subnet, Security Group, AWS IAM Role, Amazon CloudWatch và CloudWatch Alarms**. AWS IoT Core, Lambda, API Gateway, S3, SNS, ECS/ECR, Cognito, CloudFront và DynamoDB không thuộc kiến trúc hiện tại.

Bắt đầu tại [Tổng quan Workshop](5.1-Workshop-overview/).
