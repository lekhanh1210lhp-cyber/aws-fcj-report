---
title: "Thiết lập hạ tầng AWS"
date: "2026-07-28"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Tổng quan và mục tiêu

Tạo nền tảng mạng, danh tính, tài nguyên tính toán, lưu trữ và cơ sở dữ liệu cho mô hình thử nghiệm. Dùng giá trị giữ chỗ trong ghi chú và ảnh; không công khai ID tài khoản, mật khẩu, endpoint riêng hoặc khóa.

## Bước 1 - Chọn khu vực và lập kế hoạch địa chỉ

Trong AWS Console, chọn khu vực đã thống nhất của dự án. Workshop dùng **Asia Pacific (Singapore), `ap-southeast-1`** làm mốc. Ghi lại các dải CIDR không chồng lấn cho VPC, một public subnet và ít nhất hai DB subnet thuộc hai Availability Zone.

**Kết quả mong đợi:** mọi tài nguyên bên dưới nằm trong cùng khu vực và dùng tiền tố tên đã thống nhất.

## Bước 2 - Tạo hoặc chọn VPC và subnet

1. Mở **VPC → Your VPCs**, tạo hoặc chọn VPC của dự án.
2. Bật DNS resolution và DNS hostnames.
3. Tạo/chọn public subnet cho EC2.
4. Gắn Internet Gateway vào VPC.
5. Thêm `0.0.0.0/0 → Internet Gateway` vào route table của public subnet.
6. Tạo hoặc chọn hai subnet cơ sở dữ liệu ở hai Availability Zone. Không thêm route tới Internet Gateway cho các DB subnet riêng.
7. Trong **RDS → Subnet groups**, tạo DB Subnet Group chứa cả hai DB subnet.

**Kết quả mong đợi:** EC2 có thể nhận địa chỉ IPv4 công khai, còn các subnet cơ sở dữ liệu vẫn ở chế độ riêng.

## Bước 3 - Tạo Security Group

Tạo `iot-ec2-sg` và `iot-rds-sg` trong cùng VPC.

| Security Group | Loại | Nguồn | Mục đích |
| :--- | :---: | :--- | :--- |
| `iot-ec2-sg` | SSH 22 | `<ADMIN_IP>/32` | Quản trị có giới hạn |
| `iot-ec2-sg` | Custom TCP 8000 | Máy khách demo đã được duyệt | Truy cập Uvicorn trực tiếp trong buổi demo |
| `iot-ec2-sg` | HTTP 80 | Chỉ khi có reverse proxy | Không cần khi gọi trực tiếp port 8000 |
| `iot-rds-sg` | PostgreSQL 5432 | **`iot-ec2-sg`**, không phải IP CIDR | Chỉ EC2 tới RDS |

Có thể tạm dùng `0.0.0.0/0` cho cổng 8000 trong buổi demo có giám sát, nhưng đây không phải cấu hình phù hợp cho môi trường thực tế. Không đặt RDS ở chế độ công khai.

## Bước 4 - Tạo EC2 IAM Role

1. Mở **IAM → Roles → Create role**.
2. Chọn trusted entity **AWS service → EC2**.
3. Chỉ gắn `CloudWatchAgentServerPolicy` nếu CloudWatch Agent cần gửi metric/log.
4. Dùng tên trong tài liệu mã nguồn là `iot-dashboard-cloudwatch-role` (hoặc tên tương đương đã được dự án phê duyệt), sau đó tạo instance profile.

Không tạo access key dài hạn. Role cấp quyền; CloudWatch Agent được cài riêng ở mục 5.9.

## Bước 5 - Khởi tạo EC2 và cấu hình EBS

1. Khởi tạo instance từ Linux AMI đã duyệt, dùng loại instance nhỏ phù hợp với Workshop.
2. Đặt instance trong public subnet và bật IPv4 công khai để phục vụ demo.
3. Gắn `iot-ec2-sg`, key pair và IAM instance profile.
4. Cấu hình ổ đĩa gốc EBS với loại, dung lượng và chế độ mã hóa đã duyệt.
5. Thêm tag cho dự án, người sở hữu, môi trường và ngày dọn dẹp.
6. Chờ cả hai phép kiểm tra trạng thái EC2 đều đạt.

Ghi lại `<EC2_PUBLIC_IP>` nhưng không công khai khóa riêng. Địa chỉ IP công khai có thể thay đổi sau khi dừng và khởi động lại EC2; Elastic IP mới chỉ là lựa chọn trong tương lai.

## Bước 6 - Tạo Amazon RDS for PostgreSQL

1. Mở **RDS → Databases → Create database**, chọn PostgreSQL.
2. Chọn loại instance và dung lượng lưu trữ phù hợp với Workshop.
3. Đặt tên cơ sở dữ liệu ban đầu là `iot_dashboard`.
4. Chọn VPC, DB Subnet Group và `iot-rds-sg` của dự án.
5. Đặt **Public access: No**.
6. Lưu mật khẩu quản trị an toàn dưới dạng `<DB_PASSWORD>`; không đưa mật khẩu thật vào ảnh hoặc Git.
7. Chỉ bật sao lưu, mã hóa và giám sát theo cấu hình đã được dự án phê duyệt.
8. Chờ trạng thái **Available** và ghi `<RDS_ENDPOINT>`.

Không tuyên bố sử dụng Multi-AZ nếu cấu hình RDS thực tế không chứng minh điều đó.

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

Nếu thiếu `nc`, hãy cài gói netcat phù hợp với bản phân phối Linux đã chọn. Kết nối TCP thành công chỉ chứng minh route và Security Group hoạt động, chưa chứng minh thông tin đăng nhập cơ sở dữ liệu là đúng.

## Kết quả mong đợi và bằng chứng

Chụp bằng chứng đã che thông tin nhạy cảm, gồm: EC2 ở trạng thái **running**; ổ đĩa EBS và IAM Role đã gắn; RDS ở trạng thái **available**; DB Subnet Group; quy tắc của hai Security Group; và kết quả kiểm tra cổng từ EC2 tới RDS.

<!-- TODO IMAGE: /images/5-Workshop/5.4-aws-infrastructure/ec2-instance-running.png — EC2 Console hiển thị workshop instance ở trạng thái Running; che account ID, public DNS/IP khi cần và thông tin key pair. -->
<!-- TODO IMAGE: /images/5-Workshop/5.4-aws-infrastructure/ec2-iam-role-cloudwatch.png — Instance profile EC2 hiển thị iot-dashboard-cloudwatch-role và CloudWatchAgentServerPolicy; che account ID và role ARN. -->
<!-- TODO IMAGE: /images/5-Workshop/5.4-aws-infrastructure/rds-postgresql-available.png — RDS for PostgreSQL private ở trạng thái Available cùng DB Subnet Group; che endpoint và định danh tài khoản. -->
<!-- TODO IMAGE: /images/5-Workshop/5.4-aws-infrastructure/security-group-rules.png — Rule của EC2 và RDS Security Group thể hiện SSH giới hạn, cổng demo 8000 và PostgreSQL 5432 nhận từ EC2 Security Group; che public IP. -->

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| SSH hết thời gian chờ | IP công khai, route table, Internet Gateway, `<ADMIN_IP>/32` và tường lửa cục bộ |
| SSH từ chối quyền | Tài khoản đăng nhập và quyền của khóa trên máy cục bộ; không mở rộng SG để xử lý lỗi khóa |
| RDS hết thời gian chờ | Endpoint/khu vực, route của DB subnet, nguồn trong RDS SG và network ACL |
| RDS từ chối kết nối | Đúng cổng và cơ sở dữ liệu đang ở trạng thái available |
| EC2 không gửi được metric | IAM instance profile đã gắn và kết nối HTTPS đi ra |
| Trình duyệt không tới cổng 8000 | Địa chỉ bind, trạng thái dịch vụ, EC2 SG và mạng của máy khách |

Tiếp theo: [triển khai FastAPI và kết nối PostgreSQL](../5.5-Backend-and-Database/).
