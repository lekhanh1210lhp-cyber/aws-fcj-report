---
title: "Thiết lập AWS IAM & Bảo mật"
date: "2026-06-15"
weight: 1
chapter: false
pre: " <b> 5.2.1 </b> "
---

#### Tổng quan

Bảo mật là nền tảng cốt lõi khi xây dựng hệ thống Enterprise IoT Cloud Dashboard. Trước khi triển khai bất kỳ tài nguyên nào, Cloud Engineer phải thiết lập các giao thức bảo mật nghiêm ngặt.

Đảm bảo tài khoản AWS của bạn được cấu hình theo nguyên tắc đặc quyền tối thiểu. Quá trình này bao gồm việc thiết lập các người dùng (users), nhóm (groups), chính sách (policies) IAM, cũng như bắt buộc sử dụng Xác thực đa yếu tố (MFA) cho tất cả các tài khoản để ngăn chặn truy cập trái phép.

#### Cấu hình IAM

Chúng ta sẽ thực hiện thiết lập ban đầu để đảm bảo môi trường của bạn được an toàn.

Đầu tiên ở thanh tìm kiếm, truy cập vào [AWS IAM Console](https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/home).

![Truy cập vào AWS IAM](/images/5-Workshop/5.2-Prerequisite/00_AWS_IAM.jpg)

**Bước 1. Truy cập Users và Groups**

- Tại menu bên trái IAM Console, điều hướng đến **Access management**.
- Click **User groups** để thiết lập quyền truy cập dựa trên vai trò.

![Ảnh minh họa menu IAM bên trái](/images/5-Workshop/5.2-Prerequisite/01_IAM_Groups.jpg)

**Bước 2. Xác định Vai trò Nhóm**

- Click **Create group**.
- Tạo các nhóm riêng biệt cho các vai trò trong dự án: `CloudEngineers`, `BackendEngineers`, `FrontendEngineers`, và `IoTEngineers`.
- Gắn các chính sách (policies) nền tảng phù hợp cho các nhóm này (ví dụ: quyền truy cập EC2 và RDS cho Cloud Engineers).
- Click **Create group**.

![Ảnh minh họa tạo IAM Group](/images/5-Workshop/5.2-Prerequisite/02_Create_Groups.jpg)

**Bước 3. Bắt buộc sử dụng MFA**

- Điều hướng đến **Users** ở menu bên trái.
- Chọn một tài khoản người dùng cụ thể.
- Chuyển sang tab **Security credentials**.
- Trong phần Multi-factor authentication (MFA), click **Assign MFA device**.
- Làm theo hướng dẫn để cấu hình ứng dụng xác thực ảo. Đây là yêu cầu bắt buộc đối với tất cả các tài khoản.

![Ảnh minh họa cấu hình MFA](/images/5-Workshop/5.2-Prerequisite/03_Enforce_MFA.jpg)