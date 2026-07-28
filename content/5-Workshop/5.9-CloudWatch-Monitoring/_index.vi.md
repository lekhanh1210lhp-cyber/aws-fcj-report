---
title: "Giám sát bằng CloudWatch"
date: "2026-07-28"
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

# Giám sát bằng CloudWatch

## Bước 1 - Cài và cấu hình agent

Cài Amazon CloudWatch Agent trên EC2. Cấu hình agent để thu file output và error log của backend, cùng metric memory và disk. Khởi động, enable agent rồi kiểm tra trạng thái.

Dùng log stream riêng hoặc đường dẫn file rõ ràng cho:

- event của backend application; và
- error của backend.

Không ghi credential database hoặc secret trong request vào log.

## Bước 2 - Kiểm tra metric

Xác nhận có datapoint cho:

- EC2 `CPUUtilization`;
- metric memory và disk trong namespace `CWAgent`;
- RDS `CPUUtilization`; và
- RDS `DatabaseConnections`.

## Bước 3 - Tạo alarm

Tạo năm alarm cho EC2 CPU, EC2 memory, EC2 disk, RDS CPU và RDS connections. Chọn threshold phù hợp với demo instance nhỏ và ghi rõ evaluation period.

Hiểu đúng các trạng thái:

- **OK:** điều kiện không bị vi phạm;
- **In alarm:** điều kiện đã cấu hình bị vi phạm;
- **Insufficient data:** CloudWatch chưa có đủ datapoint gần đây.

SNS/email notification nằm ngoài phạm vi workshop.

**Kết quả mong đợi:** Log được gửi lên, metric có datapoint hiện tại và năm alarm hiển thị với threshold được ghi lại.

## Xử lý sự cố

- Nếu không có log, kiểm tra file path, quyền đọc, cấu hình agent, IAM Role và trạng thái agent.
- `Insufficient data` ngay sau khi tạo có thể bình thường; chờ đủ evaluation period và kiểm tra metric dimension.

Tiếp theo: [xem chi phí, bảo mật và dọn dẹp](../5.10-Cost-Security-Cleanup/).
