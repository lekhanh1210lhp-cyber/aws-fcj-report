---
title: "Giải thích về Enterprise IoT Cloud Dashboard"
date: "2026-06-15"
weight: 1
chapter: false
pre: " <b> 5.1.1 </b> "
---

#### Định nghĩa ngắn gọn

**Enterprise IoT Cloud Dashboard** là một hệ thống được xây dựng trên nền tảng AWS, thiết kế chuyên biệt cho việc quản lý tòa nhà thông minh (BMS) một cách tập trung.

Về mặt bản chất, kiến trúc của hệ thống tận dụng 5 lớp cốt lõi để vận hành:
- **Frontend**: Xây dựng bằng React.
- **Backend**: Xây dựng bằng FastAPI và chạy trên AWS EC2.
- **Database**: Cơ sở dữ liệu quan hệ sử dụng PostgreSQL RDS.
- **Monitoring**: Giám sát hệ thống thông qua CloudWatch.
- **IoT Devices**: Thiết bị biên được giả lập bởi Python Simulator.

Mục tiêu của hệ thống này là xử lý dữ liệu cảm biến một cách bảo mật và thiết lập giao tiếp hai chiều đáng tin cậy giữa bảng điều khiển từ xa và các thiết bị phần cứng như YOLO Uno hoặc ESP32.

#### Vì sao cần hệ thống này?

Các hệ thống quản lý tòa nhà truyền thống thường thiếu sự tích hợp đám mây tập trung. Dashboard này giải quyết vấn đề đó thông qua 3 khả năng chính:

- **Quản lý tập trung**: Cho phép quản lý tòa nhà thông minh (BMS) một cách tập trung thông qua hạ tầng đám mây AWS có tính sẵn sàng cao.
- **Giám sát thời gian thực**: Hỗ trợ thu thập dữ liệu viễn trắc (telemetry), cho phép hệ thống liên tục nhận thông tin về nhiệt độ, độ ẩm, ánh sáng và trạng thái thiết bị.
- **Giao tiếp hai chiều**: Cho phép quản trị viên thực thi các lệnh điều khiển từ xa trực tiếp từ Dashboard xuống thiết bị biên, ví dụ như bật/tắt quạt (Fan ON/OFF) hoặc mở rèm (Curtain OPEN).

#### Kiến trúc hoạt động

Quy trình xử lý dữ liệu viễn trắc và lệnh điều khiển từ xa diễn ra như sau:

| Bước  | Tên gọi                       | Mô tả hành động                                                                                                |
| :---- | :---------------------------- | :------------------------------------------------------------------------------------------------------------- |
| **1** | **Data Generation** (Tạo dữ liệu)     | Python Simulator tạo ra dữ liệu cảm biến thực tế, ngẫu nhiên và gửi (POST) dữ liệu đó tới FastAPI backend. |
| **2** | **Processing & Storage** (Xử lý & Lưu trữ) | FastAPI backend xác thực payload dữ liệu JSON đầu vào bằng Pydantic và lưu trữ an toàn vào PostgreSQL RDS.                   |
| **3** | **Visualization & Control** (Trực quan hóa & Điều khiển)     | React dashboard lấy dữ liệu viễn trắc để hiển thị và gửi các lệnh điều khiển từ xa (được xếp hàng trong PostgreSQL) ngược lại cho các thiết bị IoT.                     |

![IoT Architecture](/images/5-Workshop/5.1-Workshop-overview/iot-architecture-01.jpg)