---
title: "Worklog Tuần 7"
date: "2026-07-27"
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

- Triển khai trực quan hóa dữ liệu lịch sử trên Frontend Dashboard.
- Phát triển và tích hợp giao diện điều khiển thiết bị từ xa (Control Panel) cho quản trị viên.
- Đảm bảo quản trị viên có thể xem xu hướng dữ liệu và gửi lệnh trực tiếp từ giao diện (UI) xuống backend.

### Công việc thực hiện trong tuần này:

| Day | Task | Start Date | Completion Date | Reference Material |
| :-- | :--- | :--------- | :-------------- | :----------------- |
| 1   | **Chuẩn bị Trực quan hóa dữ liệu:** Đánh giá yêu cầu vẽ biểu đồ dữ liệu nhiệt độ và độ ẩm lịch sử. | 27/07/2026 | 28/07/2026 | Tài liệu Chart.js/Recharts |
| 2   | **Thiết kế Control Panel:** Đánh giá yêu cầu xây dựng các công tắc bật/tắt (toggle switches) trên giao diện. | 29/07/2026 | 29/07/2026 | Tài liệu React UI |
| 3   | **Triển khai Biểu đồ & Bảng điều khiển:** <br> - **Data Visualization:** Tích hợp thư viện Chart.js/Recharts để vẽ biểu đồ nhiệt độ và độ ẩm. <br> - **Control Panel:** Xây dựng các công tắc bật/tắt (toggle) cho Quạt, Đèn và Rèm trên UI. | 30/07/2026 | 01/08/2026 | Tài liệu Frontend Framework |
| 4   | **Gửi lệnh & Đồng bộ trạng thái:** <br> - **Command Dispatch:** Liên kết các công tắc UI với request Axios POST gửi tới endpoint `/command`. <br> - **State Synchronization:** Triển khai trạng thái "đang tải" (loading state) trên UI để chờ thiết bị IoT xác nhận. | 31/07/2026 | 01/08/2026 | Tài liệu Axios & React State |
| 5   | **Tinh chỉnh Trải nghiệm người dùng (UX):** Trau chuốt lại giao diện Dashboard để đảm bảo mang lại cảm giác của một hệ thống "Enterprise BMS" chuyên nghiệp. | 02/08/2026 | 02/08/2026 | Hướng dẫn Thiết kế UI/UX |

### Thành tựu Tuần 7:

- **Dashboard Phân tích:** Tích hợp thành công biểu đồ (Chart.js/Recharts) để trực quan hóa xu hướng dữ liệu viễn trắc lịch sử.
- **Bảng điều khiển Tương tác:** Xây dựng Control Panel hoạt động đầy đủ, cho phép admin bật/tắt Quạt, Đèn, Rèm.
- **Tích hợp API Điều khiển:** Kết nối các thành phần UI với API `/command` của backend thông qua Axios POST requests.
- **Cải thiện UX:** Triển khai trạng thái loading đồng bộ khi chờ thiết bị phản hồi và hoàn thiện giao diện chuẩn "Enterprise BMS".