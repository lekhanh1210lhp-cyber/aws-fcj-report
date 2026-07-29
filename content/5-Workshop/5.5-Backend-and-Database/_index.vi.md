---
title: "Triển khai Backend và Tích hợp Cơ sở dữ liệu"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Overview
Mục tiêu của phần này là triển khai backend FastAPI lên instance EC2, thiết lập kết nối an toàn với cơ sở dữ liệu RDS PostgreSQL, và đảm bảo dịch vụ chạy liên tục trong nền bằng `systemd`.

## Step 1 - Install the application

Kết nối tới EC2, clone repository backend, và tạo một môi trường cô lập:

```bash
git clone <BACKEND_REPOSITORY_URL> ~/aws-iot-dashboard
cd ~/aws-iot-dashboard/backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

*Lưu ý: Đảm bảo bạn đã tạo cơ sở dữ liệu (ví dụ: `iot_dashboard`) trong instance RDS của bạn ở bước Thiết lập Hạ tầng AWS (Mục 5.4). URL PostgreSQL của bạn phải chứa chính xác tên cơ sở dữ liệu này ở cuối.*

Tạo file `.env` cục bộ trên EC2 và không đưa vào Git:

```dotenv
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@<RDS_ENDPOINT>:5432/iot_dashboard
```

Kiểm tra kết nối trực tiếp tới cơ sở dữ liệu RDS bằng `psql` để xác minh kết nối mạng và security groups:

```bash
psql -h <RDS_ENDPOINT> -U <DB_USER> -d iot_dashboard
```

Chạy script khởi tạo cơ sở dữ liệu để migrate dữ liệu và tạo các bảng cần thiết:

```bash
python -m app.database.init_db
```

**Expected result:** Các thư viện phụ thuộc được cài đặt và các bảng ứng dụng được tạo thành công trong RDS.
![PostgreSQL tables](images/postgresql-tables.png)

## Step 2 - Test Uvicorn

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
curl "[http://127.0.0.1:8000/api/health](http://127.0.0.1:8000/api/health)"
```

Mở `/docs` để xem tài liệu API.

**Expected result:** Endpoint `/api/health` trả về mã trạng thái HTTP 200, và giao diện Swagger UI có thể truy cập được.
![curl health](images/curl-health.png)
![Swagger](images/swagger-ui.png)

## Step 3 - Run with systemd

Tạo `/etc/systemd/system/aws-iot-backend.service` với đúng user, working directory, environment file, và đường dẫn Uvicorn trong `.venv`. Gửi luồng output chuẩn và lỗi tới các file log đã được cấu hình của dự án, sau đó chạy:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now aws-iot-backend
sudo systemctl status aws-iot-backend
```

**Expected result:** `aws-iot-backend` đang hoạt động (active), `/api/health` trả về HTTP 200, và các bảng ứng dụng đã tồn tại trong RDS.
![systemd active](images/systemd-active.png)
![backend logs](images/backend-logs.png)

## Troubleshooting

- Sử dụng `journalctl -u aws-iot-backend -n 100` khi chạy Uvicorn thủ công thành công nhưng `systemd` bị lỗi.
- Mã hóa URL (URL-encode) các ký tự đặc biệt trong mật khẩu cơ sở dữ liệu và xác minh dịch vụ có thể đọc được file `.env`.

## Result
FastAPI backend đã được triển khai hoàn tất trên EC2, kết nối an toàn với RDS PostgreSQL, và được quản lý tin cậy bởi `systemd`. 

Tiếp theo: [kết nối YOLO UNO và các thiết bị chấp hành](../5.6-Hardware-Integration/).