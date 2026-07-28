---
title: "Kiến trúc và thiết kế dịch vụ"
date: "2026-07-28"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Kiến trúc

![Kiến trúc AWS IoT Monitoring and Control Dashboard](/images/5-Workshop/5.3-architecture/aws-iot-dashboard-architecture.png)

*Hình 5-2. Ảnh kiến trúc sao chép từ application repository thể hiện ranh giới dịch vụ, kết nối EC2-RDS, các đường HTTP của thiết bị và luồng giám sát CloudWatch.*

Người dùng dashboard, React + Vite frontend local và YOLO UNO nằm ngoài AWS. Trong AWS Cloud, VPC chứa public subnet có EC2 và DB Subnet Group có RDS. EBS là root volume của EC2. EC2 Security Group và RDS Security Group kiểm soát traffic. IAM Role thuộc AWS account, còn CloudWatch là dịch vụ regional; cả hai không nằm trong VPC.

## Thành phần và lý do chọn dịch vụ AWS

| Thành phần/dịch vụ | Trách nhiệm và lý do |
| :--- | :--- |
| React + Vite + TypeScript + Tailwind CSS | UI local cho người vận hành, telemetry, control và recommendation rule-based |
| Amazon EC2 | Toàn quyền cấu hình FastAPI, Python, Uvicorn và `systemd` |
| Amazon EBS | Root volume bền vững gắn với EC2 |
| Amazon RDS for PostgreSQL | Lưu telemetry và trạng thái command theo mô hình quan hệ |
| Amazon VPC và subnet | Ranh giới mạng cho EC2 và DB Subnet Group |
| Security Group | Rule stateful cho SSH/API và traffic EC2 tới RDS |
| AWS IAM Role | Cấp quyền tạm thời để EC2 gửi dữ liệu giám sát |
| CloudWatch Agent | Phần mềm trên EC2 đọc guest metric và file log |
| Amazon CloudWatch/Alarms | Lưu metric/log và đánh giá threshold |
| YOLO UNO / ESP32-S3 | Đọc cảm biến, điều khiển actuator, polling command và gửi ACK |

IAM Role cấp quyền; nó không phải CloudWatch Agent. Agent là process được cài đặt và quản lý trên EC2.

## API contract đã được tài liệu hóa

Source FastAPI trong `backend/main.py` và `backend/app/api/` xác nhận các route sau:

| Method | Route | Thành phần gọi |
| :--- | :--- | :--- |
| `GET` | `/` | Thông tin dịch vụ cơ bản |
| `GET` | `/api/health` | Health check |
| `POST` | `/api/telemetry` | Telemetry từ YOLO UNO |
| `GET` | `/api/devices/{device_id}/latest` | Màn hình latest của dashboard |
| `GET` | `/api/devices/{device_id}/history` | Màn hình history của dashboard |
| `POST` | `/api/devices/{device_id}/commands` | Control từ dashboard |
| `GET` | `/api/devices/{device_id}/commands/latest` | Thiết bị polling |
| `POST` | `/api/devices/{device_id}/commands/{command_id}/ack` | ACK từ thiết bị |

Firmware hỗ trợ `MODE_AUTO`, `MODE_MANUAL`, `FAN_ON`, `FAN_OFF`, `LIGHT_ON`, `LIGHT_OFF`, `CURTAIN_OPEN`, `CURTAIN_CLOSE`. Backend hiện nhận mọi chuỗi command vì `DeviceCommand` chưa có enum validator hoạt động; command không hỗ trợ sẽ giữ `Pending` vì firmware từ chối và không ACK. Không dùng dạng singular `/api/device/...`.

## Luồng dữ liệu

1. **Telemetry:** YOLO UNO gửi field camelCase → Pydantic alias map sang snake_case → SQLAlchemy ghi `telemetry_logs` trong RDS → API latest/history → dashboard.
2. **Command:** dashboard → tạo command → backend ghi `commands.state = "Pending"` → route tên `commands/latest` thực tế trả command pending cũ nhất trước (FIFO) → hardware thực thi.
3. **ACK:** thiết bị gửi command ID → backend đổi command đó thành `Executed` → telemetry sau phản ánh trạng thái actuator. ACK service hiện chỉ tìm theo command ID; kiểm tra ownership theo device là việc hardening còn lại.
4. **Monitoring:** metric mặc định EC2 và dữ liệu/log của agent → CloudWatch; RDS phát service metric; alarm đánh giá threshold đã cấu hình.

Polling có thể làm trạng thái `Pending` chỉ xuất hiện rất ngắn. Bằng chứng nên gồm response khi tạo command và trạng thái `Executed` sau đó trong database/API.

Database model định nghĩa `devices`, `telemetry_logs` và `commands`. Các field telemetry gồm `temperature`, `humidity`, `light_intensity`, `fan_status`, `light_status`, `curtain_status` và `timestamp`.

## Thiết kế mạng và bảo mật

| Nguồn | Đích | Port | Rule |
| :--- | :--- | :---: | :--- |
| `<ADMIN_IP>/32` | EC2 Security Group | 22 | Chỉ quản trị SSH |
| Client demo | EC2 Security Group | 8000 | API demo HTTP; không giữ `0.0.0.0/0` ngoài demo |
| EC2 Security Group | RDS Security Group | 5432 | Chỉ PostgreSQL |
| EC2/RDS | CloudWatch | HTTPS | Luồng outbound giám sát |

RDS không nên public. Secrets nằm trong file local đã ignore; EC2 dùng IAM Role thay cho AWS key hard-code. Thiết kế hiện tại không tuyên bố có HTTPS, authentication, HA, Multi-AZ, load balancer hoặc rate limiting.

## Khả năng mở rộng và hạn chế

Cấu trúc route theo `device_id` và schema quan hệ có thể hỗ trợ thêm phòng, nhưng phạm vi nghiệm thu hiện tại là `room_01`. Một EC2 endpoint và polling HTTP định kỳ đơn giản cho prototype nhưng có rủi ro public IP thay đổi, polling delay và một failure domain compute. Managed messaging, authentication, HTTPS, nhiều instance và Infrastructure as Code là lựa chọn tương lai, chưa phải tính năng đã triển khai.

## Kết quả mong đợi và xử lý sự cố

Mỗi mũi tên trong kiến trúc phải tương ứng với một API call, network rule, hành động database hoặc đường metric/log. Nếu một kết nối chưa rõ, hãy xác định source, destination, port, identity và bằng chứng mong đợi trước khi cấp phát.

Tiếp theo: [xây dựng hạ tầng AWS](../5.4-AWS-Infrastructure-Setup/).
