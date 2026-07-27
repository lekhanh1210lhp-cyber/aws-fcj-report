---
title: "Worklog Tuần 5"
date: "2026-07-13"
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

- Cho phép backend nhận các lệnh điều khiển (commands) từ giao diện Dashboard.
- Xây dựng cơ chế để các thiết bị IoT lấy về và thực thi các lệnh đang chờ xử lý.
- Thiết lập giao tiếp hai chiều hoàn chỉnh giữa Python Simulator và kiến trúc Cloud.

### Công việc thực hiện trong tuần này:

| Day | Task | Start Date | Completion Date | Reference Material |
| :-- | :--- | :--------- | :-------------- | :----------------- |
| 1   | **POST /command:** Tạo API endpoint để Dashboard gửi lệnh điều khiển (VD: Bật/Tắt quạt, Mở rèm). | 13/07/2026 | 14/07/2026 | Tài liệu FastAPI |
| 2   | **Command Queue:** Triển khai logic trong PostgreSQL để xếp hàng (queue) các lệnh chờ xử lý cho các thiết bị cụ thể. | 15/07/2026 | 15/07/2026 | Tài liệu PostgreSQL |
| 3   | **Device Polling:** Cập nhật Python Simulator để định kỳ lấy (GET) các lệnh chờ xử lý và xác nhận đã thực thi. | 16/07/2026 | 17/07/2026 | Tài liệu Python Requests |
| 4   | **Command Logging:** Đảm bảo tất cả các lệnh đã thực thi được ghi log trong CloudWatch để phục vụ kiểm toán (audit trails). | 18/07/2026 | 19/07/2026 | Tài liệu AWS CloudWatch |
| 5   | **System Review:** Đội ngũ đồng bộ (team sync) để đảm bảo backend hoạt động ổn định trước khi tích hợp với frontend. | 18/07/2026 | 19/07/2026 | Tài liệu Kiến trúc Hệ thống |

### Thành tựu Tuần 5:

- **Giao tiếp hai chiều:** Thiết lập thành công giao tiếp hai chiều hoàn chỉnh giữa hệ thống Simulator và Cloud.
- **Command Endpoints:** Tạo và tích hợp thành công các API endpoint cho phép điều khiển thiết bị từ xa (Quạt, Rèm).
- **Cơ chế Hàng đợi (Queue):** Thiết kế logic xếp hàng đáng tin cậy trên PostgreSQL để quản lý các lệnh IoT chờ xử lý.
- **Ghi log kiểm toán:** Tích hợp CloudWatch để duy trì dấu vết kiểm toán (audit trails) cho tất cả các lệnh đã thực thi.