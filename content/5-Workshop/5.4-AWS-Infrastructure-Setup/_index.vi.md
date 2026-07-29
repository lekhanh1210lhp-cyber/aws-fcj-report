---
title: "Thiết lập hạ tầng AWS"
date: "2026-07-28"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Tổng quan và mục tiêu

Tạo nền tảng mạng, identity, compute, storage và database cho prototype. Dùng placeholder trong ghi chú và ảnh; không công khai account ID, password, private endpoint hoặc key.

## Bước 1 - Chọn region và kế hoạch địa chỉ

Trong AWS Console, chọn region đã xác nhận của project. Workshop dùng **Asia Pacific (Singapore), `ap-southeast-1`**, làm mốc. Ghi lại các CIDR không chồng lấn cho VPC, một public subnet và tối thiểu hai DB subnet ở hai Availability Zone.

**Kết quả mong đợi:** mọi tài nguyên bên dưới nằm cùng region và dùng tiền tố tên đã thống nhất.

## Bước 2 - Tạo hoặc chọn VPC và subnet

1. Mở **VPC → Your VPCs**, tạo/chọn VPC của project.
2. Bật DNS resolution và DNS hostnames.
3. Tạo/chọn public subnet cho EC2.
4. Gắn Internet Gateway vào VPC.
5. Thêm `0.0.0.0/0 → Internet Gateway` vào route table của public subnet.
6. Tạo/chọn hai database subnet ở hai Availability Zone. Không thêm route Internet Gateway cho DB subnet private.
7. Trong **RDS → Subnet groups**, tạo DB Subnet Group chứa cả hai DB subnet.

**Kết quả mong đợi:** EC2 có thể nhận public IPv4, còn database subnet vẫn private.

## Bước 3 - Tạo Security Group

Tạo `iot-ec2-sg` và `iot-rds-sg` trong cùng VPC.

| Security Group | Loại | Nguồn | Mục đích |
| :--- | :---: | :--- | :--- |
| `iot-ec2-sg` | SSH 22 | `<ADMIN_IP>/32` | Quản trị có giới hạn |
| `iot-ec2-sg` | Custom TCP 8000 | Client demo được duyệt | Truy cập Uvicorn trực tiếp khi demo |
| `iot-ec2-sg` | HTTP 80 | Chỉ khi có reverse proxy | Không cần khi gọi trực tiếp port 8000 |
| `iot-rds-sg` | PostgreSQL 5432 | **`iot-ec2-sg`**, không phải IP CIDR | Chỉ EC2 tới RDS |

Có thể tạm dùng `0.0.0.0/0` cho port 8000 trong buổi demo có giám sát, nhưng đây không phải khuyến nghị production. Không đặt RDS thành public.

## Bước 4 - Tạo EC2 IAM Role

1. Mở **IAM → Roles → Create role**.
2. Chọn trusted entity **AWS service → EC2**.
3. Chỉ gắn `CloudWatchAgentServerPolicy` nếu CloudWatch Agent sẽ gửi metric/log.
4. Dùng tên trong runbook source là `iot-dashboard-cloudwatch-role` (hoặc tên tương đương đã được project phê duyệt) và tạo instance profile.

Không tạo access key dài hạn. Role cấp quyền; CloudWatch Agent được cài riêng ở mục 5.9.

## Bước 5 - Launch EC2 và cấu hình EBS

1. Launch Linux AMI đã duyệt và instance type nhỏ cho Workshop.
2. Đặt instance trong public subnet và bật public IPv4 cho demo.
3. Gắn `iot-ec2-sg`, key pair và IAM instance profile.
4. Cấu hình EBS root volume với loại, dung lượng và encryption đã duyệt.
5. Thêm tag project, owner, environment và ngày dọn dẹp.
6. Chờ cả hai EC2 status check pass.

Ghi lại `<EC2_PUBLIC_IP>` nhưng không công khai private key. Public IP thay đổi sau stop/start là hạn chế đã biết; Elastic IP chỉ là lựa chọn tương lai.

## Bước 6 - Tạo Amazon RDS for PostgreSQL

1. Mở **RDS → Databases → Create database**, chọn PostgreSQL.
2. Chọn instance và storage phù hợp Workshop.
3. Đặt initial database name là `iot_dashboard`.
4. Chọn VPC, DB Subnet Group và `iot-rds-sg` của project.
5. Đặt **Public access: No**.
6. Lưu master password an toàn dưới dạng `<DB_PASSWORD>`; không đưa vào ảnh hoặc Git.
7. Chỉ bật backup, encryption và monitoring đã được duyệt cho project.
8. Chờ trạng thái **Available** và ghi `<RDS_ENDPOINT>`.

Không tuyên bố Multi-AZ nếu cấu hình RDS đã triển khai không chứng minh điều đó.

## Bước 7 - Xác minh truy cập và mạng

Kết nối từ Windows PowerShell:

```powershell
ssh -i "$env:USERPROFILE\.ssh\<KEY_FILE>.pem" <EC2_USER>@<EC2_PUBLIC_IP>
```

Từ EC2 Linux Bash, kiểm tra DNS và TCP:

```bash
getent hosts <RDS_ENDPOINT>
nc -vz <RDS_ENDPOINT> 5432
```

Nếu thiếu `nc`, cài gói netcat phù hợp với Linux distribution đã chọn. TCP thành công chỉ chứng minh route và Security Group, không chứng minh database credential.

## Bằng chứng và kết quả mong đợi

Chụp bằng chứng đã che thông tin nhạy cảm về EC2 **running**, EBS volume và IAM Role đã gắn, RDS **available**, DB Subnet Group, rule của hai Security Group và kết quả test port EC2 tới RDS.

<!-- TODO IMAGE: /images/5-Workshop/5.4-aws-infrastructure/ec2-instance-running.png — EC2 Console hiển thị workshop instance ở trạng thái Running; che account ID, public DNS/IP khi cần và thông tin key pair. -->
<!-- TODO IMAGE: /images/5-Workshop/5.4-aws-infrastructure/ec2-iam-role-cloudwatch.png — Instance profile EC2 hiển thị iot-dashboard-cloudwatch-role và CloudWatchAgentServerPolicy; che account ID và role ARN. -->
<!-- TODO IMAGE: /images/5-Workshop/5.4-aws-infrastructure/rds-postgresql-available.png — RDS for PostgreSQL private ở trạng thái Available cùng DB Subnet Group; che endpoint và định danh tài khoản. -->
<!-- TODO IMAGE: /images/5-Workshop/5.4-aws-infrastructure/security-group-rules.png — Rule của EC2 và RDS Security Group thể hiện SSH giới hạn, cổng demo 8000 và PostgreSQL 5432 nhận từ EC2 Security Group; che public IP. -->

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| SSH timeout | Public IP, route table, Internet Gateway, `<ADMIN_IP>/32`, local firewall |
| SSH permission denied | Login user và quyền key local; không mở SG để xử lý lỗi key |
| RDS timeout | Endpoint/region, route DB subnet, source RDS SG, network ACL |
| RDS connection refused | Đúng port và database đang available |
| EC2 không gửi được metric | IAM instance profile và outbound HTTPS |
| Trình duyệt không tới port 8000 | Bind address, service state, EC2 SG, mạng client |

Tiếp theo: [triển khai FastAPI và kết nối PostgreSQL](../5.5-Backend-and-Database/).
