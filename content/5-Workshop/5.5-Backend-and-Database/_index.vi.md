---
title: "Triển khai backend và tích hợp cơ sở dữ liệu"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Tổng quan và mục tiêu

Cài ứng dụng Python trên Amazon Linux EC2, kết nối database private `iot_dashboard`, xác minh API theo source và duy trì Uvicorn bằng `aws-iot-backend`. Runbook ứng dụng dùng user `ec2-user`, backend path `/home/ec2-user/aws-iot-dashboard/backend`, virtual environment `venv` và entry point `main:app`.

## Bước 1 - Kết nối và cài công cụ

Từ Windows PowerShell:

```powershell
ssh -i "$env:USERPROFILE\.ssh\<KEY_FILE>.pem" ec2-user@<EC2_PUBLIC_IP>
```

Trên Amazon Linux EC2, chạy trong Linux Bash:

```bash
sudo dnf update -y
sudo dnf install -y git python3 python3-pip postgresql15 curl
```

Nếu bản Amazon Linux đã chọn dùng tên package PostgreSQL client khác, xác nhận bằng `dnf search postgresql` trước khi cài.

## Bước 2 - Clone và tạo virtual environment

Trong EC2 Linux Bash:

```bash
git clone <REPOSITORY_URL> ~/aws-iot-dashboard
cd ~/aws-iot-dashboard/backend
python3 -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Source đã kiểm tra có `backend/main.py` và export `app`; vì vậy entry point của Uvicorn là `main:app`.

## Bước 3 - Tạo environment file

Tạo `.env` đã ignore trên EC2:

```dotenv
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@<RDS_ENDPOINT>:5432/iot_dashboard
```

Nếu backend dùng `sslmode=verify-full`, tải Amazon RDS CA bundle hiện hành theo runbook của project, lưu với quyền hạn chế và dùng absolute path mà SQLAlchemy/psycopg yêu cầu:

```dotenv
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@<RDS_ENDPOINT>:5432/iot_dashboard?sslmode=verify-full&sslrootcert=<ABSOLUTE_CA_PATH>/global-bundle.pem
```

URL-encode ký tự đặc biệt trong `<DB_PASSWORD>`. Không commit `.env` hoặc password thật.

## Bước 4 - Kiểm tra PostgreSQL

Từ EC2 Linux Bash:

```bash
psql "host=<RDS_ENDPOINT> port=5432 dbname=iot_dashboard user=<DB_USER> sslmode=require"
```

Trong PostgreSQL `psql`:

```sql
SELECT current_database(), current_user;
\dt
```

Khởi tạo schema SQLAlchemy bằng lệnh có trong source:

```bash
python -m app.database.init_db
```

Lệnh gọi `Base.metadata.create_all` và dự kiến tạo `devices`, `telemetry_logs`, `commands`. Xác nhận bằng `\dt` và `\d <table_name>`; project này không định nghĩa quy trình migration Alembic.

## Bước 5 - Chạy Uvicorn thủ công

Trong EC2 Linux Bash, tại thư mục backend:

```bash
source venv/bin/activate
uvicorn main:app --host 0.0.0.0 --port 8000
```

Trong một phiên EC2 thứ hai:

```bash
curl -i http://127.0.0.1:8000/api/health
curl -s http://127.0.0.1:8000/openapi.json
```

Xác nhận tám route đã nêu ở mục 5.3 và xem Pydantic request schema được sinh ra trước khi tạo ví dụ telemetry hoặc command.

## Bước 6 - Tạo `aws-iot-backend.service`

Tạo `/etc/systemd/system/aws-iot-backend.service` bằng user, path và Uvicorn module đã xác minh:

```ini
[Unit]
Description=AWS IoT FastAPI backend
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=ec2-user
WorkingDirectory=/home/ec2-user/aws-iot-dashboard/backend
EnvironmentFile=/home/ec2-user/aws-iot-dashboard/backend/.env
ExecStart=/home/ec2-user/aws-iot-dashboard/backend/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=on-failure
RestartSec=5
StandardOutput=append:/var/log/aws-iot-backend/backend.log
StandardError=append:/var/log/aws-iot-backend/backend-error.log

[Install]
WantedBy=multi-user.target
```

Chuẩn bị thư mục log và khởi động service:

```bash
sudo install -d -o ec2-user -g ec2-user /var/log/aws-iot-backend
sudo systemctl daemon-reload
sudo systemctl enable aws-iot-backend
sudo systemctl restart aws-iot-backend
sudo systemctl status aws-iot-backend --no-pager
curl -i http://127.0.0.1:8000/api/health
```

**Kết quả mong đợi:** service là `active (running)`, health trả HTTP 200 và các table ứng dụng có trong `iot_dashboard`.

## Bước 7 - Xem log và cập nhật deployment

```bash
sudo journalctl -u aws-iot-backend -n 100 --no-pager
sudo tail -n 100 /var/log/aws-iot-backend/backend.log
cd ~/aws-iot-dashboard
git status --short
git pull --ff-only
source backend/venv/bin/activate
pip install -r backend/requirements.txt
sudo systemctl restart aws-iot-backend
curl -i http://127.0.0.1:8000/api/health
```

Chỉ pull từ branch đã duyệt, chạy lại `python -m app.database.init_db` khi model thay đổi và rà soát tương thích schema trước khi restart. Không dùng `git reset --hard` làm lối tắt triển khai.

<!-- TODO IMAGE: /images/5-Workshop/5.5-backend-database/backend-systemd-health-check.png — Terminal hiển thị aws-iot-backend active và GET /api/health trả HTTP 200; che hostname, public IP và credential. -->
<!-- TODO IMAGE: /images/5-Workshop/5.5-backend-database/postgresql-tables-and-commands.png — Bằng chứng psql cho devices, telemetry_logs, commands và query command-state đã che thông tin nhạy cảm; không để lộ RDS endpoint hoặc password. -->

## Troubleshooting

| Hiện tượng | Chẩn đoán và khắc phục |
| :--- | :--- |
| Connection refused | Xác nhận RDS/port hoặc Uvicorn đang chạy và listen |
| Connection timeout | Kiểm tra source RDS SG là `iot-ec2-sg`, subnet, endpoint và region |
| Sai `DATABASE_URL` | Kiểm tra database/user/encoding; nạp đúng `.env` mà systemd dùng |
| curl local được, remote lỗi | Bind `0.0.0.0`, kiểm tra EC2 SG port 8000 và public IP |
| `systemd` thất bại | Xem `systemctl status`, `journalctl`; kiểm tra user, path, module, quyền |
| Port đã được dùng | Chạy `sudo ss -ltnp | grep :8000` và dừng process ngoài dự kiến |
| SSL verify lỗi | Dùng đúng CA bundle, absolute path, permission và endpoint hostname |
| Thiếu table | Chạy migration/init do source định nghĩa; không tạo schema tùy ý |

Tiếp theo: [tích hợp phần cứng YOLO UNO](../5.6-Hardware-Integration/).

Tiếp theo: [kết nối YOLO UNO và các thiết bị chấp hành](../5.6-Hardware-Integration/).