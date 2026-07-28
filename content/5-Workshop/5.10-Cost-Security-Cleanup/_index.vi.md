---
title: "Chi phí, Bảo mật và Dọn dẹp"
date: "2026-07-28"
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

# Chi phí, Bảo mật và Dọn dẹp

## Kiểm tra chi phí

Ước tính trước khi triển khai bằng AWS Pricing Calculator và kiểm tra mức sử dụng thực tế bằng Billing hoặc Cost Explorer khi có thể. Bao gồm:

| Thành phần | Yếu tố chi phí chính |
|---|---|
| EC2 | Instance type và số giờ chạy |
| EBS | Dung lượng GB-tháng và snapshot |
| RDS PostgreSQL | DB class, số giờ chạy, storage, backup |
| CloudWatch | Log ingestion/storage, custom metric, alarm |

Dùng tài nguyên nhỏ cho demo, nhưng không mặc định dịch vụ miễn phí nếu chưa kiểm tra điều kiện của tài khoản và Region hiện tại.

## Checklist bảo mật

- Giới hạn SSH và cổng backend theo nguồn.
- Chỉ cho phép RDS nhận kết nối từ EC2 Security Group.
- Dùng EC2 IAM Role và policy least privilege.
- Lưu secret database và Wi-Fi ngoài Git.
- Che secret trong screenshot và log.
- Rotate mọi credential vô tình bị lộ.

## Checklist dọn dẹp

1. Xuất dữ liệu và bằng chứng cần lưu.
2. Terminate EC2 instance và xóa EBS volume/snapshot không cần thiết.
3. Xóa RDS instance và final snapshot nếu không cần.
4. Xóa CloudWatch alarm và workshop log group.
5. Xóa IAM policy/role và Security Group không dùng sau khi đã gỡ dependency.
6. Kiểm tra lại Billing/Cost Explorer.

> Dừng hoặc để lại RDS instance không có nghĩa mọi chi phí đã kết thúc. Xóa tài nguyên không còn cần và kiểm tra lại tài khoản.

**Kết quả mong đợi:** Nhóm hiểu yếu tố chi phí, bảo vệ secret và không để lại tài nguyên workshop tính phí ngoài ý muốn.

Tiếp theo: [tổng kết kết quả và bài học](../5.11-Results-Challenges-Future/).
