---
title: "Triển khai Backend và Tích hợp Cơ sở dữ liệu"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Tổng quan và mục tiêu

Cài đặt ứng dụng Python trên Amazon Linux EC2, kết nối ứng dụng với cơ sở dữ liệu riêng tư `iot_dashboard`, xác minh API được định nghĩa trong mã nguồn và duy trì Uvicorn chạy dưới dạng dịch vụ `aws-iot-backend`. Hướng dẫn triển khai ứng dụng sử dụng người dùng `ec2-user`, thư mục backend `/home/ec2-user/aws-iot-dashboard/backend`, môi trường ảo `venv` và điểm khởi chạy `main:app`.

## Bước 1 - Kết nối và cài đặt các thành phần cần thiết

Từ Windows PowerShell:

```powershell
ssh -i "$env:USERPROFILE\.ssh\<KEY_FILE>.pem" ec2-user@<EC2_PUBLIC_IP>
```

Trên Amazon Linux EC2, chạy trong Linux Bash:

```bash
sudo dnf update -y
sudo dnf install -y git python3 python3-pip postgresql15 curl
```

Nếu phiên bản Amazon Linux đã chọn sử dụng tên gói PostgreSQL client khác, hãy xác nhận bằng `dnf search postgresql` trước khi cài đặt.

## Bước 2 - Sao chép mã nguồn và tạo môi trường ảo

Trong EC2 Linux Bash:

```bash
git clone <REPOSITORY_URL> ~/aws-iot-dashboard
cd ~/aws-iot-dashboard/backend
python3 -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

*Lưu ý: Hãy đảm bảo rằng bạn đã tạo cơ sở dữ liệu (ví dụ: `iot_dashboard`). URL PostgreSQL của bạn phải chứa chính xác tên cơ sở dữ liệu này ở cuối.*

Mã nguồn đã kiểm tra có `backend/main.py` và export `app`; vì vậy điểm khởi chạy của Uvicorn là `main:app`.

## Bước 3 - Tạo tệp môi trường

Tạo tệp `.env` (được bỏ qua bởi Git) trên EC2:

```dotenv
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@<RDS_ENDPOINT>:5432/iot_dashboard
```

Nếu backend sử dụng `sslmode=verify-full`, hãy tải xuống Amazon RDS CA bundle hiện tại theo hướng dẫn của dự án, lưu tệp với quyền truy cập được giới hạn và sử dụng đường dẫn tuyệt đối theo yêu cầu của SQLAlchemy/psycopg:

```dotenv
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@<RDS_ENDPOINT>:5432/iot_dashboard?sslmode=verify-full&sslrootcert=<ABSOLUTE_CA_PATH>/global-bundle.pem
```

Mã hóa URL (URL-encode) các ký tự đặc biệt trong `<DB_PASSWORD>`. Không bao giờ commit tệp `.env` hoặc mật khẩu thực tế.

## Bước 4 - Kiểm tra PostgreSQL

Từ EC2 Linux Bash:

```bash
psql "host=<RDS_ENDPOINT> port=5432 dbname=iot_dashboard user=<DB_USER> sslmode=require"
```

Trong PostgreSQL `psql`:

```sql
SELECT current_database(), current_user;
\dt
\q
```

Thoát khỏi `psql` và khởi tạo schema SQLAlchemy bằng lệnh được định nghĩa trong mã nguồn:

```bash
python -m app.database.init_db
```

Lệnh này sẽ gọi `Base.metadata.create_all` và tạo các bảng: `devices`, `telemetry_logs` và `commands`. Xác nhận bằng `\dt` và `\d <table_name>`.

<div align="center" style="margin: 20px 0;">
  <img src="/images/5-Workshop/5.5-Backend-and-Database/hinh_9.png" alt="Successful connection to Amazon RDS" style="max-width: 100%; height: auto;">
  <p style="font-size: 1.15em; margin-top: 8px;">
    <em>Hình 8. Kết nối thành công tới Amazon RDS và xác minh bảng command trong PostgreSQL.</em>
  </p>
</div>

## Bước 5 - Chạy Uvicorn thủ công

Trong EC2 Linux Bash, từ thư mục backend:

```bash
source venv/bin/activate
uvicorn main:app --host 0.0.0.0 --port 8000
```

Trong một phiên EC2 thứ hai:

```bash
curl -i http://127.0.0.1:8000/api/health
curl -s http://127.0.0.1:8000/openapi.json
```

Xác nhận tám endpoint được mô tả trong mục 5.3 và kiểm tra các schema request Pydantic được tạo trước khi tạo ví dụ về telemetry hoặc command.

## Bước 6 - Tạo `aws-iot-backend.service`

Tạo `/etc/systemd/system/aws-iot-backend.service` bằng đúng người dùng, đường dẫn và module Uvicorn đã xác minh:

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

**Kết quả mong đợi:** dịch vụ ở trạng thái `active (running)`, endpoint health trả về HTTP 200 và các bảng của ứng dụng đã xuất hiện trong `iot_dashboard`.

<div align="center">
  <img src="/images/5-Workshop/5.5-Backend-and-Database/hinh_8.png" alt="FastAPI backend running with systemd" style="max-width: 100%;">
  <p style="font-size: 1.1em; margin-top: 8px;">
    <em>Hình 9. Backend FastAPI đang chạy dưới dạng dịch vụ systemd và trả về kết quả health check thành công.</em>
  </p>
</div>

## Bước 7 - Kiểm tra log và triển khai bản cập nhật

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

Chỉ pull từ nhánh đã được phê duyệt, chạy lại `python -m app.database.init_db` khi model thay đổi và kiểm tra khả năng tương thích của schema trước khi khởi động lại. Không sử dụng `git reset --hard` như một cách triển khai nhanh.

## Khắc phục sự cố

| Triệu chứng | Chẩn đoán và cách khắc phục |
| :--- | :--- |
| Connection refused | Xác nhận RDS/cổng hoặc kiểm tra Uvicorn đang chạy và lắng nghe kết nối |
| Connection timeout | Kiểm tra nguồn của Security Group RDS là `iot-ec2-sg`, định tuyến subnet, endpoint và region |
| Sai `DATABASE_URL` | Kiểm tra cơ sở dữ liệu/người dùng/mã hóa; nạp đúng tệp `.env` mà systemd sử dụng |
| Curl cục bộ hoạt động, truy cập từ xa thất bại | Đảm bảo bind `0.0.0.0`, kiểm tra EC2 Security Group mở cổng 8000 và địa chỉ IP công khai |
| `systemd` thất bại | Chạy `systemctl status` và `journalctl`; kiểm tra người dùng, đường dẫn, module và quyền |
| Cổng đã được sử dụng | Dùng `sudo ss -ltnp \| grep :8000` và dừng tiến trình không mong muốn |
| Xác minh SSL thất bại | Sử dụng đúng CA bundle, đường dẫn tuyệt đối, quyền truy cập và hostname của endpoint |
| Thiếu bảng | Chạy quy trình migration/khởi tạo được định nghĩa trong mã nguồn; không tự tạo schema tạm thời |

Tiếp theo: [tích hợp phần cứng YOLO UNO](../5.6-Hardware-Integration/).
