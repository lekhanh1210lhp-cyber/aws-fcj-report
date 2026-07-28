---
title: "Vận hành & Giám sát"
date: "2026-06-15"
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

#### Mục tiêu

Phần này tập trung vào việc duy trì hệ thống dashboard IoT sau khi triển khai. Mục tiêu là giúp nhóm có thể giám sát hoạt động hệ thống, phát hiện hành vi bất thường và phản ứng nhanh trước sự cố.

Chúng ta sẽ đề cập đến:

1. Giám sát sức khỏe backend bằng AWS CloudWatch.
2. Xem xét các chỉ số trên EC2 như CPU, bộ nhớ và băng thông.
3. Ghi log và kiểm toán việc thực thi lệnh để tăng tính bảo mật và dễ quản trị.

#### Quy trình giám sát

1. Mở AWS Console và điều hướng đến **CloudWatch**.
2. Xem các log group liên quan đến ứng dụng FastAPI và instance EC2.
3. Kiểm tra alarm và dashboard cho các tình trạng tăng CPU, lỗi API và khả năng phục vụ.
4. Xác nhận rằng việc thực thi lệnh và thu thập telemetry đều có thể theo dõi được trong log.

<!-- Chèn ảnh: dashboard CloudWatch hiển thị metrics và alarm -->
> Chỗ dành cho ảnh: dashboard giám sát AWS CloudWatch.

#### Kết quả mong đợi

Sau khi hoàn thành phần này, nhóm sẽ có thể quan sát tình trạng hoạt động của nền tảng và phát hiện vấn đề vận hành trước khi ảnh hưởng đến người dùng.
