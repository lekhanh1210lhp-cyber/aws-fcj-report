---
title: "[Dự kiến] Worklog Tuần 9"
date: "2026-07-27"
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9:

- Củng cố và tăng cường bảo mật cho hạ tầng đám mây.
- Đảm bảo hệ thống backend có khả năng xử lý tải trọng cấp doanh nghiệp (enterprise loads).

### Công việc thực hiện trong tuần này:

| Day | Task | Start Date | Completion Date | Reference Material |
| :-- | :--- | :--------- | :-------------- | :----------------- |
| 1   | **Kiểm toán Bảo mật (Security Audit):** Cloud Engineer đánh giá Security Groups, đảm bảo Database bị cô lập hoàn toàn khỏi public internet. | 10/08/2026 | 11/08/2026 | Các chuẩn bảo mật AWS |
| 2   | **Giới hạn tốc độ API (API Rate Limiting):** Triển khai giới hạn tỷ lệ cơ bản trong FastAPI để ngăn chặn DDoS hoặc dữ liệu viễn trắc rác (spam). | 12/08/2026 | 12/08/2026 | Tài liệu FastAPI Rate Limiting |
| 3   | **Kiểm thử chịu tải (Stress Testing):** Kỹ sư IoT cấu hình simulator để gửi dữ liệu tần số cao; giám sát tải CPU của EC2. | 13/08/2026 | 14/08/2026 | Hướng dẫn giám sát EC2 |
| 4   | **Tối ưu chi phí (Cost Optimization):** Kiểm tra cảnh báo thanh toán AWS và đảm bảo tài nguyên được định cỡ phù hợp (dùng t2.micro/t3.micro). | 15/08/2026 | 15/08/2026 | Tài liệu AWS Cost Explorer |
| 5   | **Đóng băng mã nguồn (Code Freeze):** Dừng phát triển tính năng mới; tập trung hoàn toàn vào độ ổn định của hệ thống. | 16/08/2026 | 16/08/2026 | Chiến lược Phát hành Agile |

### Thành tựu Tuần 9:

- Hạ tầng đám mây được bảo mật an toàn trước các lỗ hổng cơ bản.
- Hệ thống backend đã vượt qua kiểm thử chịu tải thành công.