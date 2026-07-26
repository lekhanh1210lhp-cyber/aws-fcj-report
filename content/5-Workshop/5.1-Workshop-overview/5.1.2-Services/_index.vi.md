---
title: "Các Dịch vụ & Công nghệ"
date: "2026-06-15"
weight: 2
chapter: false
pre: " <b> 5.1.2 </b> "
---

Kiến trúc giải pháp được xây dựng dựa trên sự phối hợp của các thành phần dịch vụ và công nghệ chính sau đây:

#### React (Frontend Dashboard)

Một ứng dụng web đáp ứng (responsive), hoạt động đầy đủ chức năng, đóng vai trò là giao diện quản lý tòa nhà thông minh tập trung.

- **Trực quan hóa dữ liệu:** Lấy và hiển thị dữ liệu viễn trắc theo thời gian thực (nhiệt độ, độ ẩm, ánh sáng) lên các UI cards, đồng thời vẽ biểu đồ dữ liệu lịch sử bằng Chart.js/Recharts.
- **Bảng điều khiển (Control Panel):** Xây dựng các công tắc bật/tắt (toggle switches) trên giao diện, cho phép quản trị viên gửi lệnh điều khiển từ xa cho quạt, đèn và rèm.
- **Điều hướng đa tòa nhà:** Triển khai thanh điều hướng (Sidebar) để chuyển đổi giữa các vị trí tòa nhà khác nhau như Hà Nội, Đà Nẵng và HCM.

#### FastAPI trên AWS EC2 (Backend)

Máy chủ backend lõi được khởi tạo bằng FastAPI và hoạt động liên tục trên một phiên bản Ubuntu EC2.

- **Thu thập & Xác thực Dữ liệu:** Nhận dữ liệu viễn trắc và triển khai các bộ xác thực (validators) bằng Pydantic để loại bỏ các payload JSON bị lỗi định dạng.
- **Điều phối Lệnh (Command Dispatching):** Cung cấp các API endpoint để Dashboard gửi lệnh điều khiển và để các thiết bị IoT truy xuất các lệnh đang chờ xử lý của chúng.
- **Bảo mật & Giới hạn tốc độ:** Triển khai tính năng giới hạn tốc độ (rate limiting) cơ bản để ngăn chặn DDoS hoặc viễn trắc rác, được bảo mật đằng sau các Security Groups định cấu hình chặt chẽ.

#### PostgreSQL trên AWS RDS (Database)

Cơ sở dữ liệu quan hệ được quản lý, triển khai bên trong một subnet nội bộ (private subnet), chỉ chấp nhận các kết nối (inbound rules) từ EC2 backend.

- **Quản lý Schema:** Sử dụng Alembic để thực hiện migrate cấu trúc dữ liệu, quản lý các bảng quan hệ cho Tòa nhà, Lịch sử Viễn trắc và Lệnh điều khiển.
- **Hàng đợi Lệnh (Command Queueing):** Triển khai logic cơ sở dữ liệu để xếp hàng (queue) các lệnh chờ xử lý một cách an toàn cho từng thiết bị biên cụ thể.
- **Tối ưu hóa Hiệu năng:** Sử dụng tính năng đánh chỉ mục (indexing) của cơ sở dữ liệu để tối ưu hóa độ trễ và thời gian phản hồi API khi truy xuất dữ liệu lịch sử.

#### Python Simulator & AWS CloudWatch (IoT & Monitoring)

Sự kết hợp giữa giả lập điện toán biên và tính năng giám sát đám mây bản địa để đảm bảo hệ thống vận hành đáng tin cậy.

- **Mô phỏng Thiết bị IoT:** Các kịch bản (scripts) Python đóng vai trò như thiết bị biên YOLO Uno hoặc ESP32, tạo ra dữ liệu cảm biến thực tế, ngẫu nhiên và sử dụng đa luồng (threading) để giả lập lưu lượng đồng thời từ nhiều tòa nhà.
- **Truy vấn Thiết bị & Đồng bộ Hai chiều:** Bộ mô phỏng định kỳ truy vấn (GET) các lệnh chờ xử lý từ backend và xác nhận đã thực thi, thiết lập giao tiếp hai chiều hoàn chỉnh.
- **Lưu vết Kiểm toán & Giám sát:** AWS CloudWatch được tích hợp để theo dõi tỷ lệ lỗi API (các phản hồi HTTP 200/500) và ghi log toàn bộ các lệnh đã thực thi nhằm phục vụ kiểm toán bảo mật.