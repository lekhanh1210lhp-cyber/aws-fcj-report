---
title: "Tổng quan Workshop"
date: "2026-07-28"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Bối cảnh và vấn đề

Các phòng nhỏ và phòng thí nghiệm thường vận hành cảm biến và thiết bị chấp hành một cách riêng lẻ. Dữ liệu không được lưu tập trung, người dùng không thể xem lại lịch sử, còn một lần nhấn nút trên dashboard chưa đủ để chứng minh thiết bị vật lý đã thực thi lệnh. Workshop giải quyết khoảng trống đó cho một phòng mẫu, nhưng không xem mô hình thử nghiệm này là hệ thống quản lý tòa nhà dùng trong môi trường thực tế.

## Đối tượng sử dụng và giải pháp đề xuất

| Đối tượng sử dụng | Nhu cầu | Giá trị từ Workshop |
| :--- | :--- | :--- |
| Người học về AWS và điện toán đám mây | Triển khai và xác minh một hệ thống AWS đầu cuối | Quy trình có thể lặp lại, bao quát từ hạ tầng AWS đến ứng dụng, phần cứng, giám sát và bằng chứng |
| Người vận hành phòng | Xem dữ liệu hiện tại, lịch sử và yêu cầu thay đổi trạng thái thiết bị chấp hành | Một dashboard để theo dõi telemetry và gửi lệnh điều khiển chế độ, quạt, đèn, rèm |
| Người bảo trì / lập trình viên | Lần theo lỗi qua phần mềm, cơ sở dữ liệu, mạng và phần cứng | Chuỗi bằng chứng liên kết API, SQL, `systemd`, Serial Monitor và CloudWatch |
| Người đánh giá dự án / mentor FCAJ | Đánh giá mức độ phù hợp với AWS, chiều sâu triển khai và đóng góp cá nhân | Quyết định kiến trúc, tiêu chí kiểm thử đo được, các đánh đổi và liên kết bàn giao rõ ràng |

YOLO UNO gửi telemetry môi trường qua HTTP tới FastAPI trên EC2. FastAPI lưu telemetry và trạng thái lệnh trong PostgreSQL. Dashboard đọc dữ liệu mới nhất, dữ liệu lịch sử và tạo lệnh; thiết bị định kỳ kiểm tra lệnh, thực thi rồi gửi ACK.

## Mức độ phù hợp với FCAJ và AWS

Workshop phù hợp với mục tiêu học tập của FCAJ vì kết hợp kiến trúc đám mây, vận hành Linux, mạng, bảo mật, cơ sở dữ liệu, phát triển full-stack, IoT vật lý, kiểm thử, giám sát, viết tài liệu và bàn giao trong một dự án có thể truy vết. AWS không chỉ là nơi chạy ứng dụng: EC2, EBS, RDS, VPC, Security Group, IAM Role, CloudWatch và CloudWatch Alarms đều đảm nhiệm một vai trò cụ thể, có cách kiểm tra vận hành, chi phí và đánh đổi kiến trúc riêng.

Phạm vi dự án phản ánh một lựa chọn kỹ thuật có cân nhắc: nhóm tiếp tục dùng thiết kế FastAPI/PostgreSQL/HTTP đã triển khai và không tuyên bố sử dụng các dịch vụ serverless hoặc IoT được quản lý khi chúng chưa có trong hệ thống. Những thành phần còn thiếu để vận hành thực tế được nêu rõ trong phần cải tiến tương lai.

## Mục tiêu kỹ thuật

1. Nhận telemetry từ phần cứng YOLO UNO thật.
2. Lấy bản ghi mới nhất và lịch sử theo thứ tự thời gian của `room_01`.
3. Điều khiển chế độ, quạt, đèn và rèm bằng tám lệnh mà firmware hỗ trợ.
4. Theo dõi quá trình hoàn tất lệnh qua trạng thái `Pending` → `Executed` và ACK.
5. Chạy backend bằng `systemd`, giám sát EC2, RDS và log.
6. Bàn giao tài liệu hướng dẫn song ngữ có thể tái tạo cùng danh sách bằng chứng cần thu thập.

## Phạm vi

| Trong phạm vi | Ngoài phạm vi triển khai hiện tại |
| :--- | :--- |
| Một thiết bị mẫu: `room_01` | BMS quy mô doanh nghiệp và vận hành nhiều đối tượng thuê |
| DHT20 đo nhiệt độ/độ ẩm | High Availability, Auto Scaling hoặc Load Balancer |
| Giá trị cảm biến ánh sáng analog thô | Lux đã hiệu chuẩn nếu firmware chưa chứng minh phép đổi |
| Quạt, đèn/relay, servo rèm | HTTPS và xác thực người dùng/thiết bị |
| FastAPI, RDS PostgreSQL, React/Vite | AWS IoT Core, Lambda, API Gateway, S3, SNS |
| EC2/EBS, VPC/SG, IAM, CloudWatch | ECS/ECR, Cognito, CloudFront, DynamoDB |

## Yêu cầu chức năng

| Chức năng | Kết quả quan sát được |
| :--- | :--- |
| Nhận telemetry | Yêu cầu hợp lệ tạo một bản ghi telemetry trong PostgreSQL |
| Telemetry mới nhất | Trả bản ghi mới nhất của `room_01` |
| Lịch sử | Trả các bản ghi của `room_01` theo thứ tự thời gian |
| Điều khiển quạt | Nhận và thực thi `FAN_ON`, `FAN_OFF` |
| Điều khiển đèn | Nhận và thực thi `LIGHT_ON`, `LIGHT_OFF` |
| Điều khiển rèm | Nhận và thực thi `CURTAIN_OPEN`, `CURTAIN_CLOSE` |
| Chế độ vận hành | `MODE_AUTO` bật điều khiển theo ngưỡng trên firmware; `MODE_MANUAL` tắt chế độ đó |
| Vòng đời lệnh | Lệnh mới có trạng thái `Pending`; ACK thành công chuyển trạng thái thành `Executed` |
| CloudWatch | Log và metric đã cấu hình được gửi đến CloudWatch; alarm đánh giá các ngưỡng |

Mã nguồn có hai cơ chế dựa trên luật, không phải mô hình AI: frontend đưa ra đề xuất theo thời gian và ngưỡng; chế độ Auto trên firmware trực tiếp bật quạt khi `temperature >= 30°C`, bật đèn khi giá trị analog `< 350` và mở rèm khi giá trị analog `< 700`. Khi nhận lệnh điều khiển trực tiếp, firmware chuyển sang chế độ Manual.

## Đầu ra cụ thể

| Đầu ra | Sản phẩm hoặc bằng chứng cụ thể |
| :--- | :--- |
| Nền tảng AWS | Danh mục tài nguyên EC2/EBS, RDS, VPC/subnet, Security Group, IAM Role và CloudWatch |
| Backend đang chạy | Dịch vụ `aws-iot-backend` ở trạng thái hoạt động, kiểm tra sức khỏe trả HTTP 200 và có mã commit đã triển khai |
| Dữ liệu PostgreSQL | Bằng chứng về bảng và câu truy vấn cho `devices`, `telemetry_logs`, `commands` |
| YOLO UNO đã tích hợp | Sơ đồ nối dây/GPIO, kết quả biên dịch PlatformIO, telemetry, quá trình thực thi lệnh và ACK trên cổng nối tiếp |
| Dashboard cục bộ | Dữ liệu mới nhất/lịch sử, yêu cầu điều khiển, ID/trạng thái lệnh trả về và nguồn dữ liệu thật/mô phỏng được phân biệt rõ |
| Giám sát | Log truy cập backend, metric EC2/RDS và năm cấu hình alarm đã được ghi nhận |
| Gói kiểm thử | Ma trận T01-T15 có trạng thái Pass/Fail/Not Run, liên kết bằng chứng, người phụ trách và kết quả chạy lại |
| Gói bàn giao | Workshop song ngữ, hạn chế đã biết, mã commit của nguồn/phiên bản triển khai và danh sách bàn giao có xác nhận |

## Tiêu chí thành công đo được

| ID | Tiêu chí | Cách đo |
| :--- | :--- | :--- |
| S01 | Backend sẵn sàng | `GET /api/health` trả HTTP 200 từ dịch vụ đã triển khai |
| S02 | Lưu telemetry | Một yêu cầu POST hợp lệ của `room_01` tạo đúng một bản ghi có thể nhận diện trong `telemetry_logs` |
| S03 | Dashboard truy xuất dữ liệu | API mới nhất và lịch sử trả đúng dữ liệu `room_01` đã lưu, theo thứ tự thời gian |
| S04 | Điều khiển vật lý | Sáu lệnh điều khiển trực tiếp đều được kiểm thử một lần, kèm ID lệnh và bằng chứng vật lý |
| S05 | Điều khiển chế độ | `MODE_AUTO` và `MODE_MANUAL` được xác minh qua hành vi firmware hoặc dữ liệu Serial Monitor |
| S06 | Hoàn tất lệnh | Cùng một ID lệnh được ghi nhận trước ở `Pending`, sau đó ở `Executed` sau ACK |
| S07 | Giám sát | Log truy cập backend, các metric EC2/RDS cần thiết và năm cấu hình alarm đều hiển thị |
| S08 | Khả năng tái tạo và an toàn | Người khác có thể làm theo tài liệu mà không thấy thông tin xác thực; mọi dòng T01-T15 đều có trạng thái |

Các tiêu chí trên là điều kiện nghiệm thu. Chỉ kết luận một phép kiểm thử đạt khi đã đính kèm đầy đủ bằng chứng tương ứng.

<!-- TODO IMAGE: /images/5-Workshop/5.1-overview/end-to-end-system-overview.png — Toàn cảnh đầu cuối gồm dashboard, backend trên EC2, trạng thái lệnh trong RDS và phần cứng YOLO UNO; không để lộ thông tin xác thực. -->

## Gợi ý xử lý sự cố

Nếu chưa xác định được thành phần gây lỗi, hãy lần theo một yêu cầu qua thẻ Network của trình duyệt, log FastAPI, PostgreSQL, Serial Monitor và trạng thái ACK. Không đánh dấu đạt khi kết quả chưa được xác minh.

Tiếp theo: [chuẩn bị tài khoản, công cụ và phần cứng](../5.2-Prerequisites/).
