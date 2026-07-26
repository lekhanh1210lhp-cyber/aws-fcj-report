---
title: "Bảo mật & Kiểm thử chịu tải"
date: "2026-06-15"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Mục tiêu

Một trong những thách thức lớn nhất đối với Enterprise IoT Cloud Dashboard là xử lý dữ liệu viễn trắc tần số cao một cách an toàn. Trước khi hoàn thiện dự án, chúng ta phải củng cố cơ sở hạ tầng đám mây và đảm bảo backend có thể xử lý tải trọng cấp doanh nghiệp.

Trong phần này, chúng ta sẽ mô phỏng kịch bản sau:

1.  Triển khai giới hạn tốc độ (rate limiting) cơ bản trong FastAPI để ngăn chặn DDoS hoặc viễn trắc rác (spam).
2.  Cấu hình IoT Simulator để gửi dữ liệu tần số cao đến EC2 backend.
3.  Giám sát CPU của EC2 và nhật ký CloudWatch để xác minh tính ổn định của hệ thống dưới tải nặng.

#### Các Bước Thực hiện

**Bước 1: Triển khai Giới hạn tốc độ API (Rate Limiting)**

Chúng ta cần xác nhận rằng FastAPI backend có thể phòng thủ trước việc bị spam request.

1.  Truy cập EC2 instance hoặc mã nguồn backend nội bộ của bạn.
2.  Cấu hình logic giới hạn tốc độ trong ứng dụng FastAPI.
    - _Ví dụ:_ Giới hạn dữ liệu viễn trắc đầu vào ở mức `100 requests / phút` cho mỗi IP.
3.  Khởi động lại dịch vụ backend (qua `systemctl`) để áp dụng các quy tắc bảo mật mới.

![Cấu hình Rate Limit](/images/5-Workshop/5.6-Stress-Testing/01_Rate_Limit.png)

**Bước 2: Cấu hình IoT Simulator để Stress Test**

Chúng ta sẽ điều chỉnh kịch bản Python để nó hoạt động như một công cụ kiểm thử chịu tải.

1.  Trên máy tính của bạn, mở dự án Python Simulator.
2.  Điều chỉnh các cấu hình luồng (threading) và độ trễ (delay) trong script để tạo ra dữ liệu tần số cao.
    ```python
    # Đoạn code ví dụ cho kiểm thử tần số cao
    import threading
    import time

    def send_spam_telemetry():
        while True:
            # Gửi POST request liên tục
            post_telemetry()
            time.sleep(0.01) # Độ trễ rất ngắn để stress test
    ```
3.  Lưu các thay đổi vào file script mô phỏng.

**Bước 3: Thực thi Kiểm thử chịu tải (Stress Test)**

Bây giờ, chúng ta sẽ "bắn phá" backend bằng dữ liệu tần số cao để xem nó phản ứng thế nào.

1.  Chạy Python Simulator đã cập nhật từ Terminal của bạn.
2.  **Quan sát kết quả trên console:**
    - Ban đầu, các request sẽ trả về `200 OK`.
    - Khi vượt quá giới hạn tốc độ, backend sẽ tự động từ chối các viễn trắc rác, trả về lỗi `429 Too Many Requests` hoặc chủ động ngắt kết nối một cách an toàn.

![Thực thi Stress Test](/images/5-Workshop/5.6-Stress-Testing/02_Stress_Test.png)

**Bước 4: Giám sát CPU EC2 & CloudWatch (Bước Xác minh)**

Hệ thống phải duy trì trạng thái ổn định bất chấp cuộc tấn công dữ liệu tần số cao.

1.  Truy cập **AWS Console**, điều hướng đến **EC2** và chọn instance backend của bạn.
2.  Chuyển sang tab **Monitoring** và xem biểu đồ **CPU Utilization**.
    - Đảm bảo CPU có tăng vọt nhưng không làm sập instance (do đã định cỡ tài nguyên phù hợp t2.micro/t3.micro).
3.  Chuyển sang **CloudWatch Console** -> Chọn **Log groups**.
4.  Xác minh rằng CloudWatch logs đã ghi nhận chính xác tất cả các API thành công (200) và các lỗi đã được xử lý (429/500).

![Giám sát CPU trên CloudWatch](/images/5-Workshop/5.6-Stress-Testing/03_CloudWatch_CPU.png)

#### Kết luận

Bạn đã thực hiện kiểm thử chịu tải và bảo mật thành công cho IoT Dashboard!

- **Hạ tầng được Bảo mật:** Backend đã được bảo vệ trước các lỗ hổng DDoS cơ bản và dữ liệu viễn trắc rác.
- **Sẵn sàng tải trọng Doanh nghiệp:** Hệ thống đã xử lý dữ liệu tần số cao một cách hiệu quả.
- **Đóng băng Mã nguồn (Code Freeze):** Với tính ổn định đã được xác minh, bạn hiện đã sẵn sàng cho việc Code Freeze và bàn giao dự án.