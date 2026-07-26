---
title: "Kiểm thử REST API (Thu thập Dữ liệu)"
date: "2026-06-15"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Mục tiêu

Sau khi triển khai thành công FastAPI backend và kết nối với PostgreSQL, đã đến lúc xác minh luồng dữ liệu. Trong phần này, bạn sẽ đóng vai trò là bộ mô phỏng IoT, gửi các payload viễn trắc thủ công thông qua Postman trực tiếp tới EC2 backend để quan sát cách API thu thập dữ liệu hoạt động.

Chúng ta sẽ tập trung vào 2 yếu tố:

1.  **Thu thập dữ liệu (Data Ingestion):** Backend có nhận và lưu trữ thành công dữ liệu viễn trắc vào PostgreSQL không?
2.  **Xác thực dữ liệu (Data Validation):** Hệ thống có áp dụng chính xác các bộ xác thực Pydantic để từ chối các JSON payload bị lỗi định dạng không?

#### Các Bước Thực hiện

**Bước 1: Cấu hình môi trường Postman**

Để bắt đầu kiểm thử, chúng ta cần thiết lập môi trường gửi request nhắm mục tiêu đến FastAPI trên EC2 instance.

1.  Mở ứng dụng Postman trên máy tính của bạn.
2.  Tạo một Request mới bằng cách nhấp vào nút **"+"** hoặc chọn **New > HTTP Request**.
3.  Cấu hình phương thức HTTP thành **POST**.
4.  Nhập URL Request trỏ đến IP công khai hoặc Elastic IP của EC2 (ví dụ: `http://<EC2-Elastic-IP>:8000/telemetry`).

![Mở môi trường Postman](/images/5-Workshop/5.4-Test-REST-API/01_Postman_Setup.jpg)

**Bước 2: Gửi Payload Viễn trắc (Trường hợp Hợp lệ)**

Bây giờ, hãy thử gửi một JSON payload hợp lệ mô phỏng dữ liệu nhiệt độ, độ ẩm và ánh sáng.

1.  Trong request Postman, chuyển sang tab **Body**.
2.  Chọn **raw** và đảm bảo định dạng được đặt là **JSON**.
3.  Nhập payload dữ liệu JSON hợp lệ:
    ```json
    {
      "building_id": "HN_01",
      "temperature": 25.4,
      "humidity": 60,
      "light": 450,
      "device_status": "active"
    }
    ```
4.  Nhấp **Send**.
5.  **Quan sát kết quả:**
    - Backend sẽ xử lý và xác thực dữ liệu bằng Pydantic.
    - Nó sẽ trả về trạng thái `200 OK` hoặc `201 Created` kèm theo thông báo thành công xác nhận đã lưu vào PostgreSQL.

![Kiểm thử Payload Hợp lệ](/images/5-Workshop/5.4-Test-REST-API/02_Valid_Payload.jpg)

**Bước 3: Kiểm thử Xác thực Dữ liệu (Trường hợp Lỗi)**

Đây là tính năng quan trọng để đảm bảo tính ổn định của hệ thống: từ chối các payload sai trước khi chúng tới được cơ sở dữ liệu.

1.  Chỉnh sửa JSON payload của bạn để bao gồm kiểu dữ liệu không hợp lệ hoặc thiếu các trường bắt buộc. Ví dụ, truyền một chuỗi chữ (string) vào trường nhiệt độ:
    ```json
    {
      "building_id": "HN_01",
      "temperature": "too_hot",
      "humidity": 60
    }
    ```
2.  Nhấp **Send**.
3.  **Quan sát kết quả:**
    - Bộ xác thực Pydantic sẽ chặn request này lại.
    - Hệ thống sẽ trả về lỗi `422 Unprocessable Entity`.
    - Bạn sẽ thấy một thông báo lỗi chi tiết chỉ ra cụ thể các trường nào không vượt qua bước xác thực.

![Kiểm thử Payload Bị lỗi](/images/5-Workshop/5.4-Test-REST-API/03_Malformed_Payload.jpg)

**Bước 4: Xác minh qua CloudWatch Logs**

Để đảm bảo hệ thống được giám sát toàn diện, chúng ta cần xác minh qua các bản ghi nhật ký (logs) của backend.

1.  Truy cập AWS Console và điều hướng đến **CloudWatch**.
2.  Tìm Log Group liên kết với dịch vụ backend EC2 của bạn.
3.  **Kết quả mong đợi:**
    - Bạn sẽ thấy các bản ghi xác nhận các lần gọi API thành công cũng như tỷ lệ lỗi do các request sai định dạng gây ra.
    - Điều này đảm bảo hệ thống có khả năng quan sát (visibility) đầy đủ về tỷ lệ lỗi API theo đúng yêu cầu dự án.

![Xác minh bằng CloudWatch](/images/5-Workshop/5.4-Test-REST-API/04_CloudWatch_Verify.jpg)