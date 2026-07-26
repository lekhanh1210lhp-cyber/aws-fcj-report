---
title: "Chuẩn bị môi trường"
date: "2026-06-15"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Mục tiêu

Trước khi xây dựng Enterprise IoT Cloud Dashboard, chúng ta cần thiết lập một nền tảng vững chắc. Giống như việc chuẩn bị nguyên liệu trước khi nấu ăn, phần này đảm bảo rằng môi trường AWS của bạn đã sẵn sàng với các giao thức bảo mật cần thiết và kiến trúc cốt lõi.

Trong phần này, chúng ta sẽ hoàn thành 3 mục tiêu khởi tạo quan trọng:

1.  **Thiết lập môi trường AWS:** Chốt sơ đồ kiến trúc 5 lớp và phân công vai trò nghiêm ngặt cho các thành viên trong nhóm.
2.  **Thiết lập giao thức bảo mật IAM:** Cloud Engineer tiến hành tạo IAM users, groups, policies và bắt buộc sử dụng MFA cho tất cả các tài khoản để đảm bảo an toàn.
3.  **Khởi tạo Repository:** Thiết lập Git repository và xác định các chiến lược phân nhánh (branching strategies) theo tiêu chuẩn Agile/Scrum.

#### Các thành phần chính

Trong phần chuẩn bị này, chúng ta sẽ tương tác với các thành phần sau:

- **AWS IAM (Identity and Access Management):** Dịch vụ được sử dụng để thiết lập người dùng, nhóm, chính sách và bắt buộc sử dụng xác thực đa yếu tố (MFA).
- **Amazon VPC (Virtual Private Cloud):** Dịch vụ mạng nền tảng, nơi chúng ta sẽ thiết kế VPC, các mạng con (subnets) public/private, Internet Gateway và Route Tables.
- **Git Repository:** Hệ thống quản lý mã nguồn được khởi tạo để quy chuẩn chiến lược phân nhánh và chuẩn bị cho các kịch bản triển khai sau này.

#### Các Bước Thực hiện

1. [Thiết lập AWS IAM & Bảo mật](5.2.1-IAM-Setup/)
2. [Cấu hình VPC & Networking](5.2.2-VPC-Networking/)