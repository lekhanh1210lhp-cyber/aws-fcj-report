---
title: "Triển khai Backend và Tích hợp Cơ sở dữ liệu"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Triển khai Backend và Tích hợp Cơ sở dữ liệu

## Bước 1 - Cài ứng dụng

Kết nối EC2, clone backend repository và tạo môi trường độc lập:

```bash
git clone <BACKEND_REPOSITORY_URL> ~/aws-iot-dashboard
cd ~/aws-iot-dashboard/backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Tạo `.env` trực tiếp trên EC2 và không đưa file vào Git:

```dotenv
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@<RDS_ENDPOINT>:5432/iot_dashboard
```

Chạy lệnh migration hoặc khởi tạo bảng của project, sau đó kiểm tra các bảng bằng `psql`.

## Bước 2 - Kiểm tra Uvicorn

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000
curl http://127.0.0.1:8000/api/health
```

Chỉ mở `/openapi.json` hoặc `/docs` từ nguồn được phép.

## Bước 3 - Chạy bằng systemd

Tạo `/etc/systemd/system/aws-iot-backend.service` với user, working directory, environment file và đường dẫn Uvicorn trong `.venv` chính xác. Chuyển standard output và error tới các file log được project cấu hình, sau đó chạy:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now aws-iot-backend
sudo systemctl status aws-iot-backend
```

**Kết quả mong đợi:** `aws-iot-backend` active, `/api/health` trả HTTP 200 và các bảng ứng dụng tồn tại trong RDS.

## Xử lý sự cố

- Dùng `journalctl -u aws-iot-backend -n 100` khi Uvicorn chạy thủ công nhưng `systemd` thất bại.
- URL-encode ký tự đặc biệt trong password database và kiểm tra service có thể đọc `.env`.

Tiếp theo: [kết nối YOLO UNO và actuator](../5.6-Hardware-Integration/).
