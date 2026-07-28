---
title: "Kiến trúc và Thiết kế Dịch vụ"
date: "2026-07-28"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Kiến trúc và Thiết kế Dịch vụ

React client và YOLO UNO nằm ngoài AWS. FastAPI chạy trên EC2 trong public subnet; PostgreSQL chạy trên RDS qua DB Subnet Group. EC2 sử dụng IAM Role để gửi dữ liệu vận hành tới CloudWatch mà không hard-code AWS access key.

![Kiến trúc AWS IoT Dashboard](/images/2-Proposal/IoT_Dashboard_Architecture.png)

## Luồng dữ liệu

1. **Telemetry:** YOLO UNO → `POST /api/telemetry` → FastAPI → RDS → màn hình latest/history trên dashboard.
2. **Command:** dashboard → tạo command → RDS có row ở trạng thái `Pending`.
3. **Thực thi và ACK:** YOLO UNO polling command mới nhất, điều khiển actuator rồi gửi ACK; FastAPI cập nhật row thành `Executed`.
4. **Quan sát hệ thống:** file log backend → CloudWatch Agent → CloudWatch Logs; metric EC2/RDS/CWAgent → CloudWatch alarm.

## Lý do chọn dịch vụ và ranh giới bảo mật

- **EC2:** chủ động quản lý FastAPI, Uvicorn, `systemd` và file log.
- **RDS for PostgreSQL:** lưu trữ quan hệ được quản lý và duy trì lịch sử telemetry/command.
- **EBS:** lưu hệ điều hành, ứng dụng và log local của EC2.
- **CloudWatch:** tập trung log, metric và alarm.
- **IAM Role:** cung cấp credential AWS tạm thời cho workload EC2.

Giới hạn SSH theo public IP của nhóm, chỉ mở cổng backend cho nguồn được phép và cấu hình RDS Security Group chỉ nhận PostgreSQL (`5432`) từ EC2 Security Group.

**Kết quả mong đợi:** Mỗi thành phần, ranh giới tin cậy và luồng dữ liệu đều có mục đích rõ ràng.

## Xử lý sự cố

- Nếu thiết bị không kết nối được EC2, kiểm tra public IP, route, Network ACL và EC2 Security Group.
- Nếu EC2 không kết nối được RDS, kiểm tra endpoint, port, DB Subnet Group và rule nguồn theo Security Group.

Tiếp theo: [khởi tạo hạ tầng AWS](../5.4-AWS-Infrastructure-Setup/).
