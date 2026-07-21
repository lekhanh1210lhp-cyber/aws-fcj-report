# Enterprise IoT Cloud Dashboard on AWS 🚀

Hệ thống quản lý tập trung các tòa nhà thông minh đa chi nhánh thông qua nền tảng Cloud (Building Management System - BMS thu nhỏ).

---

## 📋 Tổng quan dự án

Dự án này giải quyết bài toán thực tế khi một doanh nghiệp sở hữu nhiều tòa nhà văn phòng tại các khu vực khác nhau (Hà Nội, Đà Nẵng, TP.HCM) và cần một giải pháp giám sát tập trung toàn bộ thông số cảm biến (nhiệt độ, độ ẩm, ánh sáng) cũng như điều khiển thiết bị từ xa (quạt, đèn, rèm) mà không cần can thiệp thủ công tại từng chi nhánh.

- **Tác giả:** Lê Bảo Khánh
- **Đơn vị:** Trường Đại học Bách Khoa TP.HCM (HCMUT) - Khoa Khoa học và Kỹ thuật Máy tính

---

## 🏛️ Kiến trúc hệ thống (5 Layers)

Toàn bộ hệ thống được phân rã thành các tầng rõ ràng, đảm bảo tính tách biệt và khả năng mở rộng quy mô:
┌──────────────────────────────────────────┐
│              React Dashboard             │
└──────────────────────────────────────────┘
▲
│ REST API
▼
┌──────────────────────────────────────────┐
│         FastAPI Backend (AWS EC2)        │
└──────────────────────────────────────────┘
│              │
│              │
▼              ▼
PostgreSQL         CloudWatch
(RDS)             Logs
▲
│
│ HTTP
▼
┌──────────────────────────────────────────┐
│         Python Simulator / YOLO Uno      │
└──────────────────────────────────────────┘

### Chi tiết các thành phần:
1. **IoT Devices / Simulator:** Đọc thông số cảm biến (nhiệt độ, độ ẩm, ánh sáng) và gửi bản tin telemetry định kỳ lên Cloud (hiện mô phỏng bằng Python, có thể thay thế bằng phần cứng thực tế như YOLO Uno, ESP32, Arduino mà không cần sửa đổi Backend).
2. **AWS EC2 (Backend):** Đóng vai trò là "bộ não" chạy FastAPI, chịu trách nhiệm tiếp nhận request, xác thực, xử lý logic, ghi log qua CloudWatch và tương tác với Database.
3. **PostgreSQL (Database):** Lưu trữ dữ liệu chuỗi thời gian (Telemetry), lịch sử hoạt động và trạng thái lệnh điều khiển thiết bị.
4. **React Dashboard:** Giao diện quản trị tập trung cung cấp biểu đồ trực quan, thống kê và bảng điều khiển thời gian thực cho người vận hành.
5. **AWS CloudWatch:** Giám sát hệ thống, theo dõi log hoạt động và đảm bảo khả năng vận hành ổn định của Backend trên Cloud.

---

## 🔄 Luồng hoạt động chính

- **Chiều lên (Telemetry):** Thiết bị đo đạc thông số ➔ Gửi HTTP POST request lên FastAPI ➔ Lưu trữ vào PostgreSQL ➔ Hiển thị lên React Dashboard.
- **Chiều xuống (Command):** Người quản trị thao tác trên Dashboard ➔ Gửi lệnh điều khiển qua FastAPI ➔ Thiết bị nhận lệnh thực thi (bật/tắt quạt, đèn, rèm) và cập nhật trạng thái mới.

---

## 👥 Phân chia vai trò thành viên trong nhóm

- **Member 1 (Cloud Engineer):** Quản lý hạ tầng AWS, EC2, CloudWatch, IAM và tiến hành Deploy hệ thống.
- **Member 2 (Backend Engineer):** Phát triển FastAPI, xây dựng REST API và thiết kế cơ sở dữ liệu PostgreSQL.
- **Member 3 (Frontend Engineer):** Xây dựng giao diện React Dashboard, tích hợp hệ thống biểu đồ và bảng điều khiển.
- **Member 4 (IoT Engineer):** Viết Python Simulator, thực hiện kiểm thử thiết bị và tài liệu hóa phần cứng (sẵn sàng tích hợp YOLO Uno / ESP32).

---

## 💡 Tại sao lựa chọn hạ tầng AWS?
- **Quản lý tập trung:** Tập trung dữ liệu từ tất cả các tòa nhà phân tán trên một nền tảng duy nhất.
- **Khả năng mở rộng:** Dễ dàng thêm các chi nhánh mới hoặc tích hợp thiết bị phần cứng mới mà không làm gián đoạn hệ thống cũ.
- **Tối ưu vận hành:** Loại bỏ chi phí và phức tạp khi phải tự duy trì server vật lý tại từng địa điểm.

---

## 📄 License & Bản quyền
Copyright © 2026 Lê Bảo Khánh - HCMUT. All rights reserved.