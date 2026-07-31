---
title: "Workshop"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

Workshop này hướng dẫn xây dựng một quy trình đầu cuối cho thiết bị có mã `room_01`: YOLO UNO đọc dữ liệu từ DHT20 và cảm biến ánh sáng analog; FastAPI lưu telemetry cùng lệnh điều khiển vào Amazon RDS for PostgreSQL; React hiển thị dữ liệu và gửi lệnh; sau khi điều khiển quạt, đèn hoặc rèm, thiết bị gửi ACK để xác nhận đã thực thi.

## Mục tiêu và kết quả cuối

Sau Workshop, bạn có thể:

- triển khai backend FastAPI trên Amazon EC2 với ổ đĩa gốc Amazon EBS;
- kết nối EC2 với Amazon RDS for PostgreSQL qua mạng riêng trong DB Subnet Group;
- tích hợp quá trình gửi telemetry, thăm dò lệnh, thực thi lệnh và gửi ACK trên YOLO UNO;
- kết nối dashboard React + Vite + TypeScript chạy trên máy cục bộ;
- xác minh lệnh chuyển từ `Pending` sang `Executed`; và
- thu thập bằng chứng vận hành của EC2, RDS và backend trên CloudWatch.

Kết quả cuối cùng là một mô hình thử nghiệm có thể tái tạo cho `room_01`, không phải hệ thống quản lý tòa nhà ở quy mô doanh nghiệp. Sau khi chuẩn bị sẵn mã nguồn và tài khoản AWS, người học dự kiến cần khoảng **8–12 giờ** để hoàn thành.

## Nội dung Workshop

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

![Kiến trúc AWS IoT Monitoring and Control Dashboard](/images/5-Workshop/5.3-architecture/aws_diagram.png)

*Hình 5-1. Sơ đồ kiến trúc từ kho mã nguồn ứng dụng: người dùng dashboard, React frontend chạy cục bộ và YOLO UNO nằm ngoài AWS; EC2 và RDS nằm trong VPC; IAM và CloudWatch hoạt động ở cấp tài khoản hoặc khu vực nên không nằm trong ranh giới VPC.*

Kiến trúc hiện sử dụng **Amazon EC2, Amazon EBS, Amazon RDS for PostgreSQL, Amazon VPC, subnet, Security Group, AWS IAM Role, Amazon CloudWatch và CloudWatch Alarms**. AWS IoT Core, Lambda, API Gateway, S3, SNS, ECS/ECR, Cognito, CloudFront và DynamoDB chưa được sử dụng.

Bắt đầu tại [Tổng quan Workshop](5.1-Workshop-overview/).
