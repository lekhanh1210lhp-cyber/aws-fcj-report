---
title: "Kiểm thử REST API (Thu thập Dữ liệu)"
date: "2026-06-15"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Mục tiêu

Sau khi backend FastAPI đã kết nối với PostgreSQL, việc tiếp theo là xác minh rằng các request thực tế có thể được chấp nhận, kiểm tra và lưu trữ đúng cách. Phần này mô phỏng vai trò của các thiết bị IoT khi gửi dữ liệu viễn trắc tới API.

Chúng ta sẽ tập trung vào hai khía cạnh chính:

1. **Thu thập dữ liệu:** Xác nhận các request hợp lệ được chấp nhận và lưu vào database.
2. **Xác thực dữ liệu:** Đảm bảo các payload thiếu thông tin hoặc sai kiểu dữ liệu bị từ chối với thông báo lỗi rõ ràng.

#### Chuẩn bị

Trước khi kiểm thử, hãy đảm bảo backend đang chạy trên EC2 và kết nối tới database đã hoạt động bình thường. Endpoint API cần có thể truy cập được từ máy local.

<!-- Chèn ảnh: thiết lập request Postman đến endpoint EC2 -->
> Chỗ dành cho ảnh: Postman đã được cấu hình để gọi đến endpoint telemetry.

#### Bước 1: Gửi payload hợp lệ

Sử dụng Postman để gửi một request mẫu.

1. Mở Postman và tạo một request mới.
2. Đặt phương thức là **POST**.
3. Nhập endpoint EC2, ví dụ `http://<EC2-Elastic-IP>:8000/telemetry`.
4. Trong tab Body, chọn **raw** và **JSON**, sau đó gửi payload sau:

```json
{
  "building_id": "HN_01",
  "temperature": 25.4,
  "humidity": 60,
  "light": 450,
  "device_status": "active"
}
```

Kết quả mong đợi: backend sẽ trả về `200 OK` hoặc `201 Created`, và dữ liệu sẽ được ghi vào database.

#### Bước 2: Gửi payload không hợp lệ

Để kiểm tra cơ chế kiểm tra dữ liệu, hãy gửi một request có trường sai kiểu dữ liệu.

```json
{
  "building_id": "HN_01",
  "temperature": "too_hot",
  "humidity": 60
}
```

Kết quả mong đợi: service sẽ từ chối request với mã `422 Unprocessable Entity` và trả về lỗi xác thực.

#### Bước 3: Xác minh trong CloudWatch

Sau khi gửi cả hai request, hãy xem logs backend trong CloudWatch.

1. Mở AWS Console và điều hướng đến **CloudWatch**.
2. Mở log group của backend EC2.
3. Xác nhận các request thành công và các lỗi validation đã được ghi nhận.

<!-- Chèn ảnh: logs CloudWatch hiển thị request thành công và bị từ chối -->
> Chỗ dành cho ảnh: giao diện log của CloudWatch cho các request telemetry.

#### Kết luận

Bước này giúp xác thực hành vi cốt lõi của API IoT: chấp nhận dữ liệu viễn trắc đúng, từ chối dữ liệu sai và để lại dấu vết rõ ràng trong hệ thống giám sát.