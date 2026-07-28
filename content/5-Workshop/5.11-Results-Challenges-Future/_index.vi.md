---
title: "Kết quả, Thách thức và Hướng phát triển"
date: "2026-07-28"
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

# Kết quả, Thách thức và Hướng phát triển

## Kết quả

Luồng đã triển khai kết nối telemetry cảm biến, lịch sử PostgreSQL bền vững, React dashboard, điều khiển thiết bị vật lý, command ACK và khả năng quan sát bằng CloudWatch. Chỉ công bố kết quả khi có bằng chứng tương ứng từ Mục 5.8.

## Thách thức và cách khắc phục

| Thách thức | Cách khắc phục |
|---|---|
| Endpoint singular/plural không khớp | Theo OpenAPI contract và dùng một canonical path |
| Uvicorn chạy nhưng `systemd` lỗi | Kiểm tra user, working directory, environment file và `journalctl` |
| Vite proxy hoặc CORS lỗi | Dùng relative URL `/api/...` và kiểm tra proxy target |
| EC2 public IP thay đổi | Cập nhật cấu hình; chỉ dùng endpoint ổn định khi chủ động cấp phát |
| Command thực thi lặp | Lưu last command ID và chỉ ACK sau khi thành công |
| CloudWatch Agent không có dữ liệu | Kiểm tra IAM Role, path, permission, dimension và agent status |

## Hạn chế

Workshop dùng một EC2 instance, HTTP trong quá trình phát triển, chưa có application authentication và không bảo đảm public IP tĩnh. Vì vậy, tài liệu không tuyên bố high availability, bảo mật cấp production hoặc cam kết latency chưa đo.

## Hướng phát triển

- HTTPS và managed domain;
- authentication và authorization;
- cấu hình triển khai ổn định và CI/CD;
- thiết kế backend/database có khả năng scale theo nhu cầu đã đo; và
- automation nâng cao sau khi baseline rule-based an toàn đã được xác thực.

**Kết quả mong đợi:** Kết quả, hạn chế và hướng phát triển có bằng chứng hỗ trợ và dùng thuật ngữ kỹ thuật chính xác.

Tiếp theo: [chuẩn bị bàn giao dự án](../5.12-Project-Handover/).
