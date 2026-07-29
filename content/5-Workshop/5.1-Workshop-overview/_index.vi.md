---
title: "Tổng quan Workshop"
date: "2026-07-28"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Bối cảnh và vấn đề

Các phòng nhỏ và phòng thí nghiệm thường vận hành cảm biến, thiết bị chấp hành riêng lẻ. Dữ liệu không được lưu tập trung, người dùng không xem được lịch sử và một lần nhấn trên dashboard chưa chứng minh thiết bị vật lý đã thực thi. Workshop giải quyết khoảng trống đó cho một phòng mẫu, không mô tả prototype như hệ thống quản lý tòa nhà production.

## Người dùng và giải pháp đề xuất

Người dùng chính gồm người triển khai Workshop, người vận hành xem dashboard và người bảo trì điều tra lỗi. YOLO UNO gửi telemetry môi trường qua HTTP tới FastAPI trên EC2. FastAPI lưu telemetry và trạng thái command trong PostgreSQL. Dashboard đọc dữ liệu latest/history và tạo command; thiết bị polling, thực thi rồi gửi ACK.

## Mục tiêu kỹ thuật

1. Nhận telemetry từ phần cứng YOLO UNO thật.
2. Lấy bản ghi mới nhất và lịch sử theo thời gian của `room_01`.
3. Điều khiển chế độ, quạt, đèn và rèm bằng tám command firmware hỗ trợ.
4. Quan sát được việc hoàn tất command qua `Pending` → `Executed` và ACK.
5. Chạy backend bằng `systemd`, giám sát EC2, RDS và log.
6. Bàn giao runbook song ngữ có thể tái tạo và checklist bằng chứng.

## Phạm vi

| Trong phạm vi | Ngoài phạm vi triển khai hiện tại |
| :--- | :--- |
| Một thiết bị mẫu: `room_01` | BMS enterprise và vận hành multi-tenant |
| DHT20 đo nhiệt độ/độ ẩm | High Availability, Auto Scaling hoặc Load Balancer |
| Giá trị cảm biến ánh sáng analog thô | Lux đã hiệu chuẩn nếu firmware chưa chứng minh phép đổi |
| Quạt, đèn/relay, servo rèm | HTTPS và authentication |
| FastAPI, RDS PostgreSQL, React/Vite | AWS IoT Core, Lambda, API Gateway, S3, SNS |
| EC2/EBS, VPC/SG, IAM, CloudWatch | ECS/ECR, Cognito, CloudFront, DynamoDB |

## Contract chức năng

| Chức năng | Kết quả quan sát được |
| :--- | :--- |
| Nhận telemetry | Request hợp lệ tạo một bản ghi telemetry trong PostgreSQL |
| Telemetry mới nhất | Trả bản ghi mới nhất của `room_01` |
| Lịch sử | Trả các bản ghi của `room_01` theo thứ tự thời gian |
| Điều khiển quạt | Nhận và thực thi `FAN_ON`, `FAN_OFF` |
| Điều khiển đèn | Nhận và thực thi `LIGHT_ON`, `LIGHT_OFF` |
| Điều khiển rèm | Nhận và thực thi `CURTAIN_OPEN`, `CURTAIN_CLOSE` |
| Chế độ vận hành | `MODE_AUTO` bật điều khiển threshold trên firmware; `MODE_MANUAL` tắt chế độ đó |
| Vòng đời command | Command mới là `Pending`; ACK thành công đổi thành `Executed` |
| CloudWatch | Log/metric đã cấu hình đến CloudWatch và alarm đánh giá threshold |

Source có hai cơ chế rule-based, không phải mô hình AI: frontend tạo recommendation theo thời gian/threshold, còn Auto mode trên firmware trực tiếp điều khiển quạt khi `temperature >= 30°C`, đèn khi giá trị analog `< 350` và rèm quanh threshold `< 700`. Command actuator trực tiếp sẽ chuyển firmware sang Manual mode.

## Tiêu chí thành công và thành phẩm

Workshop thành công khi telemetry tới RDS, dashboard đọc được dữ liệu, từng command được hỗ trợ chỉ thực thi một lần, ACK cập nhật trạng thái lưu trữ, CloudWatch nhận bằng chứng đã cấu hình và người khác có thể làm lại quy trình mà không thấy credential thật trong tài liệu.

Thành phẩm gồm tài nguyên AWS, backend service đã triển khai, bản ghi database, firmware đã cấu hình, dashboard local, bằng chứng kiểm thử, màn hình CloudWatch, Workshop song ngữ và checklist bàn giao.

<!-- TODO IMAGE: /images/5-Workshop/5.1-overview/end-to-end-system-overview.png — Toàn cảnh end-to-end gồm dashboard, EC2 backend, RDS command state và phần cứng YOLO UNO, không để lộ credential. -->

## Điểm kiểm tra sự cố

Nếu nhóm chưa xác định được component gây lỗi, hãy theo dấu một request qua Network của trình duyệt, FastAPI log, PostgreSQL, Serial Monitor và trạng thái ACK. Không đánh dấu pass cho kết quả chưa được xác minh.

Tiếp theo: [chuẩn bị tài khoản, công cụ và phần cứng](../5.2-Prerequisites/).
