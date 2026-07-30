---
title: "Triển khai backend và tích hợp cơ sở dữ liệu"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Tổng quan và mục tiêu

Triển khai ứng dụng Python trên EC2 Amazon Linux, kết nối tới cơ sở dữ liệu riêng `iot_dashboard`, đối chiếu API với mã nguồn và chạy Uvicorn ổn định bằng dịch vụ `aws-iot-backend`. Quy trình vận hành sử dụng tài khoản `ec2-user`, thư mục backend `/home/ec2-user/aws-iot-dashboard/backend`, môi trường ảo `venv` và điểm vào `main:app`.

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

Nếu phiên bản Amazon Linux đã chọn dùng tên gói PostgreSQL client khác, hãy kiểm tra bằng `dnf search postgresql` trước khi cài.

## Bước 2 - Sao chép kho mã nguồn và tạo môi trường ảo

Trong EC2 Linux Bash:

```bash
git clone <REPOSITORY_URL> ~/aws-iot-dashboard
cd ~/aws-iot-dashboard/backend
python3 -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Qua kiểm tra, file `backend/main.py` có xuất đối tượng `app`; vì vậy điểm vào Uvicorn là `main:app`.

## Bước 3 - Tạo file biến môi trường

Tạo file `.env` trên EC2 và bảo đảm file này đã được loại khỏi Git:

```dotenv
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@<RDS_ENDPOINT>:5432/iot_dashboard
```

Nếu backend dùng `sslmode=verify-full`, hãy tải gói chứng chỉ CA hiện hành của Amazon RDS theo tài liệu dự án, lưu file với quyền truy cập hạn chế và dùng đường dẫn tuyệt đối theo yêu cầu của SQLAlchemy/psycopg:

```dotenv
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@<RDS_ENDPOINT>:5432/iot_dashboard?sslmode=verify-full&sslrootcert=<ABSOLUTE_CA_PATH>/global-bundle.pem
```

Nếu `<DB_PASSWORD>` chứa ký tự đặc biệt, hãy mã hóa các ký tự đó theo chuẩn URL. Không commit file `.env` hoặc mật khẩu thật.

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

Khởi tạo lược đồ SQLAlchemy bằng lệnh được định nghĩa trong mã nguồn:

```bash
python -m app.database.init_db
```

Lệnh này gọi `Base.metadata.create_all` để tạo ba bảng `devices`, `telemetry_logs`, `commands`. Hãy xác nhận bằng `\dt` và `\d <table_name>`; dự án hiện chưa có quy trình migration bằng Alembic.

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

Đối chiếu tám route đã nêu ở mục 5.3 và xem lược đồ yêu cầu do Pydantic tạo ra trước khi gửi telemetry hoặc lệnh mẫu.

## Bước 6 - Tạo `aws-iot-backend.service`

Tạo `/etc/systemd/system/aws-iot-backend.service` bằng tài khoản, đường dẫn và module Uvicorn đã xác minh:

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

Chuẩn bị thư mục log và khởi động dịch vụ:

```bash
sudo install -d -o ec2-user -g ec2-user /var/log/aws-iot-backend
sudo systemctl daemon-reload
sudo systemctl enable aws-iot-backend
sudo systemctl restart aws-iot-backend
sudo systemctl status aws-iot-backend --no-pager
curl -i http://127.0.0.1:8000/api/health
```

**Kết quả mong đợi:** dịch vụ ở trạng thái `active (running)`, API kiểm tra sức khỏe trả HTTP 200 và các bảng ứng dụng đã có trong `iot_dashboard`.

## Bước 7 - Xem log và cập nhật phiên bản triển khai

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

Chỉ cập nhật từ nhánh đã được duyệt. Khi mô hình dữ liệu thay đổi, hãy chạy lại `python -m app.database.init_db` và kiểm tra tính tương thích của lược đồ trước khi khởi động lại dịch vụ. Không dùng `git reset --hard` như một cách triển khai nhanh.

## Kết quả mong đợi

Dịch vụ `aws-iot-backend` ở trạng thái `active (running)`, `GET /api/health` trả HTTP 200, EC2 kết nối được với RDS riêng và ba bảng `devices`, `telemetry_logs`, `commands` tồn tại trong `iot_dashboard`. Bằng chứng triển khai phải ghi mã commit của ứng dụng và không chứa thông tin xác thực.

<!-- TODO IMAGE: /images/5-Workshop/5.5-backend-database/backend-systemd-health-check.png — Terminal hiển thị aws-iot-backend đang hoạt động và GET /api/health trả HTTP 200; che tên máy, địa chỉ IP công khai và thông tin xác thực. -->
<!-- TODO IMAGE: /images/5-Workshop/5.5-backend-database/postgresql-tables-and-commands.png — Bằng chứng psql cho các bảng devices, telemetry_logs, commands và truy vấn trạng thái lệnh đã che thông tin nhạy cảm; không để lộ endpoint RDS hoặc mật khẩu. -->

## Xử lý sự cố

| Hiện tượng | Chẩn đoán và khắc phục |
| :--- | :--- |
| Bị từ chối kết nối | Xác nhận đúng RDS và cổng kết nối; nếu gọi API, kiểm tra Uvicorn đang chạy và lắng nghe |
| Hết thời gian chờ | Kiểm tra nguồn của RDS SG là `ec2-rds-1`, subnet, endpoint và khu vực |
| Sai `DATABASE_URL` | Kiểm tra tên cơ sở dữ liệu, người dùng, cách mã hóa; nạp đúng `.env` mà `systemd` sử dụng |
| curl cục bộ thành công, truy cập từ xa thất bại | Bind `0.0.0.0`, kiểm tra cổng 8000 trong EC2 SG và IP công khai |
| `systemd` không khởi động được | Xem `systemctl status`, `journalctl`; kiểm tra tài khoản, đường dẫn, module và quyền |
| Cổng đã được sử dụng | Chạy `sudo ss -ltnp | grep :8000` và dừng tiến trình ngoài dự kiến |
| Xác minh SSL thất bại | Dùng đúng CA bundle, đường dẫn tuyệt đối, quyền file và tên máy chủ endpoint |
| Thiếu bảng | Chạy lệnh khởi tạo hoặc migration do mã nguồn định nghĩa; không tự tạo một lược đồ khác |

Tiếp theo: [tích hợp phần cứng YOLO UNO](../5.6-hardware-integration/).
