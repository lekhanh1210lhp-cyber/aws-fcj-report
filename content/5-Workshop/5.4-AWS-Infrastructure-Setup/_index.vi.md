---
title: "Thiết lập Hạ tầng AWS"
date: "2026-07-28"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Thiết lập Hạ tầng AWS

## Bước 1 - Network và Security Group

Trong `ap-southeast-1`, tạo hoặc chọn VPC có public subnet cho EC2 và ít nhất hai subnet phù hợp cho RDS DB Subnet Group.

Tạo:

- `iot-ec2-sg`: SSH (`22`) từ public IP của bạn và cổng backend (`8000`) chỉ từ các client IP được phép.
- `iot-rds-sg`: PostgreSQL (`5432`) với nguồn là `iot-ec2-sg`.

Không mở RDS cho `0.0.0.0/0`.

## Bước 2 - IAM Role và EC2

Tạo IAM Role cho EC2, gắn `CloudWatchAgentServerPolicy`, sau đó launch một Linux instance nhỏ trong public subnet. Gắn `iot-ec2-sg`, IAM Role và EBS volume phù hợp cho demo.

## Bước 3 - RDS PostgreSQL

Tạo PostgreSQL instance với:

- database name: `iot_dashboard`;
- kết nối private;
- DB Subnet Group; và
- `iot-rds-sg`.

Trên EC2, cài PostgreSQL client và kiểm tra:

```bash
psql "host=<RDS_ENDPOINT> port=5432 dbname=iot_dashboard user=<DB_USER> sslmode=require"
```

**Kết quả mong đợi:** `psql` mở được PostgreSQL session từ EC2 trong khi truy cập database trực tiếp từ Internet vẫn bị chặn.

## Xử lý sự cố

- Timeout thường liên quan route hoặc Security Group; lỗi authentication liên quan endpoint, user, password hoặc database name.
- Nếu EC2 thiếu quyền CloudWatch, kiểm tra IAM Role đã được gắn vào instance chứ không chỉ mới được tạo.

Tiếp theo: [triển khai backend và kết nối cơ sở dữ liệu](../5.5-Backend-and-Database/).
