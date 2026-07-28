---
title: "Điều kiện tiên quyết"
date: "2026-07-28"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Mục tiêu

Chuẩn bị tài khoản AWS, công cụ local, phần cứng, thông tin truy cập và kiến thức nền trước khi tạo tài nguyên có tính phí. Ví dụ dùng Asia Pacific (Singapore), `ap-southeast-1`, làm mốc cho Workshop; hãy xác nhận region thực tế và giữ EC2, RDS, VPC, CloudWatch trong cùng region đó.

## Quyền truy cập AWS

- Tài khoản AWS có quyền xem billing và đã bật MFA.
- Quyền least privilege cho EC2, EBS, RDS, VPC, Security Group, gắn IAM Role, CloudWatch và CloudWatch Alarms.
- Quyền pass IAM Role đã được duyệt cho EC2. Workshop không yêu cầu AdministratorAccess.
- EC2 key pair lưu local; tuyệt đối không commit `.pem` hoặc `.key`.
- Biết public IP của quản trị viên để tạo rule SSH: `<ADMIN_IP>/32`.

Trước khi cấp phát, ghi lại region, VPC CIDR, public subnet CIDR, hai DB subnet ở hai Availability Zone, tiền tố tên tài nguyên và người chịu trách nhiệm dọn dẹp.

## Công cụ local và kiểm tra version

Chạy từng lệnh trong đúng môi trường được ghi. Có thể dùng version mới hơn nếu source project hỗ trợ.

| Công cụ | Môi trường và lệnh kiểm tra | Kết quả mong đợi |
| :--- | :--- | :--- |
| Git | PowerShell: `git --version` | Một version Git |
| Python | PowerShell: `python --version` | Version tương thích FastAPI `0.128.8` và SQLAlchemy `2.0.46` |
| Node.js | PowerShell: `node --version` | Version tương thích Vite `8.1.1` và TypeScript `6.0.2` |
| npm | PowerShell: `npm --version` | Một version npm dạng số |
| PlatformIO | PlatformIO terminal: `pio --version` | Version PlatformIO Core |
| PostgreSQL client | PowerShell: `psql --version` | Version PostgreSQL client |
| Trình duyệt | Mở DevTools → Network | Có thể kiểm tra request |
| VS Code | Help → About | Hiện version đã cài |

Nếu PowerShell không tìm thấy công cụ, mở lại terminal sau khi cài và chạy `Get-Command <tool>`. Không trộn `%USERPROFILE%` của Command Prompt với `$env:USERPROFILE` của PowerShell.

## Phần cứng và an toàn điện

Chuẩn bị:

- YOLO UNO / ESP32-S3 và cáp USB data;
- cảm biến nhiệt độ, độ ẩm DHT20;
- cảm biến ánh sáng analog;
- module quạt và driver phù hợp khi cần;
- đèn hoặc module relay;
- servo motor điều khiển rèm;
- màn hình LCD1602 I2C có trong firmware cuối;
- dây nối, breadboard và nguồn đúng công suất.

Không cấp nguồn cho motor hoặc tải relay trực tiếp từ GPIO. Xác minh mức điện áp, driver/diode bảo vệ và nối chung ground trước khi kết nối actuator.

## Mức sẵn sàng của source và secrets

Source ứng dụng là repository riêng `aws-iot-dashboard`. Repository có `requirements.txt` của backend, `package.json` của frontend, `platformio.ini`, `boards/yolo_uno.json` và `include/secrets.example.h`. `.gitignore` đã loại trừ `.env`, `.pem`, virtual environment, build output và `node_modules`; cần xác nhận `hardware/include/secrets.h` vẫn untracked trước khi chia sẻ.

Source đã review xác nhận Uvicorn `main:app`, user Amazon Linux `ec2-user`, virtual environment `venv`, các bảng `devices`, `telemetry_logs`, `commands` và GPIO số được ghi ở mục 5.6.

## Checklist kiến thức

Người học nên hiểu REST method/status code, truy vấn PostgreSQL, tham chiếu Security Group, FastAPI/OpenAPI, React/Vite, PlatformIO, quyền Linux cơ bản và `systemd`.

## Cổng sẵn sàng

- [ ] Đã thống nhất region và quy tắc đặt tên.
- [ ] Quyền AWS least privilege hoạt động.
- [ ] Biết `<ADMIN_IP>` và vị trí EC2 key.
- [ ] Hoàn tất mọi lệnh kiểm tra version.
- [ ] Đã rà soát yêu cầu nguồn của phần cứng.
- [ ] Có source repository và template secret đã ignore.
- [ ] Đã phân công người dọn dẹp và nơi lưu evidence.

<!-- TODO IMAGE: /images/5-Workshop/5.2-prerequisites/development-tools-versions.png — Bằng chứng terminal hiển thị version Git, Python, Node.js, npm, PlatformIO và psql đã xác minh. -->

## Kết quả và xử lý sự cố

Chỉ tiếp tục khi mọi mục chặn ở trên đã hoàn tất. Nếu thiếu source repository hoặc pin map firmware chính xác, ghi nhận là blocker bàn giao; không tự đặt giá trị và không cấp nguồn cho mạch.

Tiếp theo: [xem kiến trúc và ranh giới dịch vụ](../5.3-Architecture-and-Service-Design/).
