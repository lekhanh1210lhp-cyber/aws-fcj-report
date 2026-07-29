---
title: "Bàn giao dự án"
date: "2026-07-28"
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

## Tổng quan và mục tiêu

Chuyển giao đủ source, cấu hình, kiến thức vận hành và evidence để người bảo trì mới có thể khởi động, xác minh, cập nhật, xử lý sự cố và dọn dẹp prototype an toàn.

## Cấu trúc repository

Gói bàn giao ứng dụng cần có:

```text
<application-repository>/
├── backend/              # FastAPI, Pydantic, SQLAlchemy, requirements
├── frontend/             # React, Vite, TypeScript, Tailwind CSS
├── hardware/             # PlatformIO firmware and YOLO UNO board definition
│   └── include/
│       └── secrets.example.h
└── README.md
```

Source ứng dụng đã review được quản lý riêng tại `F:\aws-iot-dashboard`; repository này chứa báo cáo Hugo và Workshop. Ghi lại:

- source repository: `<SOURCE_REPOSITORY_URL>`;
- demo video: `<VIDEO_DEMO_URL>`;
- commit ứng dụng đã triển khai: `<COMMIT_SHA>`;
- vị trí AWS region/resource inventory: `<HANDOVER_EVIDENCE_LOCATION>`.

## Quy trình khởi động

Backend trên EC2 Linux Bash:

```bash
sudo systemctl start aws-iot-backend
sudo systemctl status aws-iot-backend --no-pager
curl -i http://127.0.0.1:8000/api/health
```

Frontend trên Windows PowerShell:

```powershell
Set-Location .\aws-iot-dashboard\frontend
npm install
npm run dev
```

Hardware trong PlatformIO terminal:

```bash
pio run
pio run --target upload
pio device monitor --baud 115200
```

## Local secret cần thiết

| Vị trí | Giá trị | Quy tắc lưu |
| :--- | :--- | :--- |
| Backend `.env` | `DATABASE_URL` và setting do source định nghĩa | Chỉ EC2/local; hạn chế quyền; ignore |
| Firmware `secrets.h` | Wi-Fi, API URL, `room_01` | Chỉ local; ignore |
| Frontend `.env.local` nếu dùng | API base URL | Chỉ local; ignore |
| EC2 key | Private key | Kho secret local đã duyệt; không đưa vào Git |

Bàn giao quy trình lấy/rotate secret, không ghi credential plaintext trong báo cáo.

## Checklist AWS và vận hành

- [ ] Biết đúng AWS account và region.
- [ ] Đã ghi VPC, public subnet, DB Subnet Group, route table và tag.
- [ ] Đã ghi EC2, EBS, key owner, IAM Role và `iot-ec2-sg`.
- [ ] Đã ghi RDS identifier/endpoint, database `iot_dashboard` và `iot-rds-sg`.
- [ ] RDS vẫn private, port 5432 nhận từ EC2 SG.
- [ ] `aws-iot-backend` và CloudWatch Agent chạy khi boot.
- [ ] Đã ghi backend log group, metric dimension, retention và alarm.
- [ ] Đã ghi firmware build `room_01`, GPIO map chính xác và yêu cầu nguồn an toàn.
- [ ] Đã link kết quả T01-T15 mới nhất và issue đang mở.
- [ ] Đã phân công cost owner và ngày dọn dẹp.

## Quy trình cập nhật deployment

Trong EC2 Linux Bash:

```bash
cd ~/aws-iot-dashboard
git status --short
git pull --ff-only
source backend/venv/bin/activate
pip install -r backend/requirements.txt
cd backend
python -m app.database.init_db
cd ..
sudo systemctl restart aws-iot-backend
sudo systemctl status aws-iot-backend --no-pager
curl -i http://127.0.0.1:8000/api/health
```

Review thay đổi model/schema và release note trước. `app.database.init_db` dùng SQLAlchemy `create_all`, không phải migration engine; thay đổi schema destructive hoặc không tương thích cần quy trình riêng đã review. Ghi commit cũ/mới và rollback procedure. Không bỏ local change bằng `git reset --hard`.

## Kiểm tra database và CloudWatch

Từ EC2 Linux Bash:

```bash
psql "host=<RDS_ENDPOINT> port=5432 dbname=iot_dashboard user=<DB_USER> sslmode=require"
sudo systemctl status amazon-cloudwatch-agent --no-pager
sudo tail -n 100 /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

Trong `psql`, chạy `\dt`, xem `devices`, `telemetry_logs`, `commands` và dùng query read-only để xác minh. Trong CloudWatch, kiểm tra region, cả hai backend log group, timestamp mới, guest metric `IoTDashboard/EC2`, EC2 CPU native, RDS CPU/connections và tên/state của sáu alarm đã tài liệu hóa.

## Hạn chế đã biết

Prototype dùng một phòng, HTTP trực tiếp port 8000 cho demo, public IP EC2 có thể đổi, polling định kỳ và giá trị ánh sáng analog chưa hiệu chuẩn. Hệ thống chưa triển khai HTTPS, route authentication/API-key enforcement, HA, bằng chứng Multi-AZ, load balancer, rate limiting hoặc AI model. Frontend có thể dùng simulation và báo mock success sau lỗi, lưu mode local, gọi sai ánh sáng là Lux và hard-code EC2 target; backend thiếu command enum validation và kiểm tra ownership ACK chặt. GPIO, source path/schema và threshold alarm đề xuất đã ghi ở 5.5, 5.6, 5.9 nhưng vẫn cần evidence xác nhận môi trường đang chạy.

## Trách nhiệm nhóm

| Thành viên | Trách nhiệm |
| :--- | :--- |
| **Phạm Lê Minh Khôi** | Kiến trúc AWS, VPC, Security Group, IAM Role, EC2, RDS, CloudWatch, DevOps, phần cứng YOLO UNO, cảm biến, actuator, telemetry, command polling, ACK |
| **Ngô Minh Thuận** | FastAPI backend, endpoint, Pydantic schema, SQLAlchemy model, PostgreSQL integration, telemetry processing, command lifecycle, ACK processing |
| **Thượng Đình Hưng** | React + Vite frontend, dashboard UI, telemetry visualization, control, tích hợp tổng thể, debug, quay/dựng demo video |
| **Lê Bảo Khánh** | Tài liệu, proposal, blog, weekly worklog, event report, Workshop, review song ngữ, navigation, screenshot, quality assurance |

## Checklist bàn giao cuối

- [ ] Người nhận mở được source link và đúng commit ID.
- [ ] Không có credential trong Git, ảnh, video hoặc Workshop.
- [ ] Đã demo cách khởi động backend, frontend và firmware.
- [ ] Đã review OpenAPI route và database schema từ source.
- [ ] Đã bàn giao GPIO map số và power diagram.
- [ ] Test matrix có actual evidence và status.
- [ ] Đã xác nhận cấu hình CloudWatch và alarm threshold.
- [ ] Đã sign-off issue, hạn chế, owner, quyết định chi phí và trạng thái dọn dẹp.

<!-- TODO IMAGE: /images/5-Workshop/5.12-handover/repository-handover-checklist.png — Checklist bàn giao repository/resource/test cuối đã che thông tin nhạy cảm, có commit ID, owner, open issue và xác nhận của nhóm. -->

Quay lại [trang Workshop](../).
