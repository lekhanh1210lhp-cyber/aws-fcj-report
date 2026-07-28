---
title: "Bảo mật & Kiểm thử chịu tải"
date: "2026-06-15"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Mục tiêu

Một trong những thách thức lớn nhất đối với Enterprise IoT Cloud Dashboard là xử lý dữ liệu viễn trắc tần số cao một cách an toàn. Trước khi hoàn thiện dự án, chúng ta cần củng cố hạ tầng và đảm bảo backend vẫn ổn định dưới tải trọng tương đương môi trường doanh nghiệp.

Trong phần này, chúng ta sẽ mô phỏng các tình huống sau:

1. Áp dụng giới hạn tốc độ API để giảm thiểu spam và tấn công kiểu DDoS.
2. Tăng tần suất gửi telemetry để kiểm tra backend dưới tải.
3. Giám sát chỉ số EC2 và logs CloudWatch để xác nhận hệ thống vẫn hoạt động bình thường.

#### Bước 1: Áp dụng giới hạn tốc độ

Chúng ta sẽ cấu hình FastAPI để từ chối các request vượt quá ngưỡng cho phép từ cùng một nguồn.

1. Mở mã nguồn backend và thêm middleware hoặc dependency-based rate limiting.
2. Đặt ngưỡng thực tế như `100 requests/phút` cho mỗi IP.
3. Khởi động lại dịch vụ FastAPI để các quy tắc có hiệu lực.

<!-- Chèn ảnh: cấu hình rate limiting trong FastAPI -->
> Chỗ dành cho ảnh: giao diện cấu hình giới hạn tốc độ trong backend.

#### Bước 2: Tạo lưu lượng cao

Một script Python đơn giản có thể được dùng để tạo lưu lượng liên tục.

```python
import time

while True:
    post_telemetry()
    time.sleep(0.01)
```

Điều này giúp nhóm quan sát cách API phản ứng khi lưu lượng tăng đột ngột.

#### Bước 3: Thực hiện stress test

Chạy script mô phỏng và quan sát phản hồi của hệ thống.

1. Khởi động simulator từ terminal.
2. Quan sát việc request ban đầu trả về `200 OK` rồi chuyển sang `429 Too Many Requests` sau khi vượt ngưỡng.
3. Xác nhận ứng dụng vẫn có thể phục vụ và ghi log đầy đủ.

<!-- Chèn ảnh: output terminal hoặc kết quả Postman trong quá trình stress test -->
> Chỗ dành cho ảnh: kết quả kiểm thử chịu tải từ simulator.

#### Bước 4: Quan sát chỉ số AWS

Bước xác minh cuối cùng là kiểm tra tầng giám sát.

1. Mở instance EC2 trong AWS Console.
2. Xem tab **Monitoring** để kiểm tra CPU utilization và network activity.
3. Mở CloudWatch và xác nhận logs phản ánh cả request thành công lẫn các sự kiện quá tải được xử lý.

<!-- Chèn ảnh: dashboard CloudWatch hoặc biểu đồ EC2 monitoring -->
> Chỗ dành cho ảnh: CloudWatch metrics cho CPU và lỗi API.

#### Kết luận

Bước này cho thấy hệ thống có khả năng chịu tải tốt và xác nhận backend cùng tầng giám sát đã sẵn sàng cho triển khai thực tế.