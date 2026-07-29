---
title: "Chi phí, bảo mật và dọn dẹp"
date: "2026-07-28"
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

## Tổng quan và mục tiêu

Hiểu các yếu tố tạo chi phí, rà soát ranh giới bảo mật của mô hình thử nghiệm, lưu bằng chứng cần thiết và xóa tài nguyên theo đúng thứ tự phụ thuộc. Không nêu giá cố định vì chi phí phụ thuộc khu vực, kích thước tài nguyên, thời gian lưu giữ, lưu lượng truyền dữ liệu và bảng giá hiện hành.

## Bước 1 - Rà soát các yếu tố tạo chi phí

| Tài nguyên | Yếu tố tạo chi phí | Cách kiểm soát trong Workshop |
| :--- | :--- | :--- |
| Amazon EC2 | Loại instance và số giờ chạy | Dùng instance nhỏ đã duyệt; dừng hoặc terminate sau khi sử dụng |
| Amazon EBS | Loại ổ đĩa, dung lượng cấp phát và snapshot | Chọn dung lượng phù hợp; xóa volume/snapshot không còn gắn |
| Amazon RDS for PostgreSQL | DB class, thời gian chạy, lưu trữ và sao lưu | Dùng cơ sở dữ liệu nhỏ; xóa sau khi thu đủ bằng chứng nếu được duyệt |
| Amazon CloudWatch | Metric tùy chỉnh, lượng log thu nhận/lưu trữ và alarm | Chỉ thu metric hệ điều hành mỗi 60 giây khi cần; đặt thời gian lưu ngắn |
| Truyền dữ liệu | Lưu lượng giữa người dùng/thiết bị và EC2 | Chọn chu kỳ gửi telemetry và thăm dò hợp lý |

Dùng AWS Pricing Calculator hoặc hóa đơn thực tế để lập ước tính có ghi ngày. Không đưa mức giá chưa xác minh vào báo cáo.

## Bước 2 - Rà soát ranh giới bảo mật

- Dùng danh tính theo nguyên tắc đặc quyền tối thiểu và bật MFA.
- Dùng EC2 IAM Role; không ghi cố định AWS access key trong mã nguồn.
- Giới hạn SSH port 22 theo `<ADMIN_IP>/32`.
- Chỉ cho RDS 5432 nhận từ EC2 Security Group.
- Giữ RDS private.
- Loại `.env`, `.pem`, `.key` và `hardware/include/secrets.h` khỏi Git.
- Dùng placeholder trong tài liệu và che thông tin nhạy cảm trên ảnh.
- Xem việc mở HTTP cổng 8000 công khai là một hạn chế của bản demo.
- Rà soát quy tắc đi ra, thời gian lưu log, người dùng cơ sở dữ liệu và tag tài nguyên.
- Thay mới ngay mọi thông tin bí mật nếu chúng xuất hiện trong Git, lịch sử terminal, ảnh hoặc video demo.

Các khuyến nghị trước khi dùng trong môi trường thực tế gồm reverse proxy, HTTPS, xác thực, phân quyền, quản lý bí mật, sao lưu và thiết kế mạng đã được rà soát. Những tính năng này chưa được triển khai trong phiên bản hiện tại.

## Bước 3 - Lưu bằng chứng trước khi dọn dẹp

Trước khi xóa, lưu:

1. sơ đồ kiến trúc và danh mục tài nguyên;
2. trạng thái EC2/dịch vụ và mã commit đã triển khai;
3. bằng chứng bảng/câu truy vấn RDS không chứa thông tin xác thực;
4. kết quả kiểm thử telemetry, lệnh và ACK;
5. ảnh CloudWatch log/metric/alarm; và
6. bằng chứng demo phần cứng/frontend.

Xác nhận người sở hữu snapshot và thời gian phải lưu bằng chứng.

## Bước 4 - Chỉ dọn dẹp tài nguyên thuộc dự án

1. Dừng lưu lượng telemetry/lệnh mới và đưa thiết bị chấp hành về trạng thái an toàn.
2. Lưu bằng chứng và quyết định về snapshot cuối đã ghi ở Bước 3.
3. Dừng rồi terminate EC2 của dự án sau khi được phê duyệt bàn giao.
4. Kiểm tra thiết lập `DeleteOnTermination` của ổ đĩa gốc EBS; chỉ xóa volume không còn gắn và snapshot thực sự thuộc dự án.
5. Xóa RDS for PostgreSQL của dự án; chỉ tạo snapshot cuối khi có yêu cầu lưu giữ và đã được phê duyệt.
6. Xóa sáu CloudWatch alarm của dự án và, sau khi quyết định thời gian lưu, xóa hai log group của dự án.
7. Xóa `iot-dashboard-cloudwatch-role` và instance profile chỉ khi không có hệ thống nào khác sử dụng.
8. Xóa `iot-ec2-sg` và `iot-rds-sg` sau khi không còn ENI hoặc tài nguyên phụ thuộc.
9. Nếu Workshop đã tạo riêng DB Subnet Group, public subnet, hai DB subnet, route table, Internet Gateway và VPC, hãy xóa theo thứ tự phụ thuộc. Không xóa tài nguyên mạng có sẵn hoặc dùng chung.
10. Mở Billing/Cost Explorer và danh mục tài nguyên theo tag để xác minh không còn tài nguyên dự án ngoài dự kiến.

Cách triển khai hiện tại không tạo S3 bucket và không có stack/state của CloudFormation, SAM, CDK hoặc Terraform. Vì vậy, quy trình dọn dẹp **không** yêu cầu xóa bucket hoặc stack. Nếu phiên bản sau bổ sung các tài nguyên này, phải cập nhật danh mục và thứ tự phụ thuộc trước.

Dừng RDS chỉ là biện pháp tạm thời và chịu giới hạn của dịch vụ; RDS có thể tự khởi động lại. Muốn tránh chi phí dài hạn, cần xóa cơ sở dữ liệu và các tài nguyên tính phí khác theo quyết định lưu giữ của nhóm.

## Bước 5 - Xác minh kết quả dọn dẹp

- Kiểm kê lại tài nguyên theo tag trong đúng khu vực.
- Kiểm tra instance đã dừng, volume EBS không còn gắn, snapshot RDS được giữ lại, log group và alarm không còn sử dụng.
- Nếu không xóa được Security Group, hãy tìm ENI hoặc tài nguyên phụ thuộc thay vì cưỡng chế xóa.
- Nếu không xóa được cơ sở dữ liệu, rà soát chế độ bảo vệ xóa và yêu cầu snapshot với người sở hữu.
- Ghi ID của từng tài nguyên đã xóa trong bằng chứng dọn dẹp; không để lộ thông tin xác thực.

<!-- TODO IMAGE: /images/5-Workshop/5.10-cleanup/aws-resource-inventory.png — Resource inventory AWS trước/sau và kiểm tra Billing/Cost Explorer đã che thông tin nhạy cảm, thể hiện kết quả dọn dẹp Workshop. -->

## Kết quả mong đợi

Bằng chứng cần thiết được giữ lại; không còn tài nguyên Workshop thuộc dự án phát sinh phí ngoài phê duyệt; tài nguyên dùng chung không bị ảnh hưởng; phần rà soát bảo mật phản ánh đúng hạn chế hiện tại và không tuyên bố hệ thống đã sẵn sàng cho môi trường thực tế.

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| Không xóa được Security Group | Tìm ENI, EC2, RDS hoặc Security Group tham chiếu còn phụ thuộc |
| Không xóa được VPC/subnet | Kiểm tra Internet Gateway, route table association, DB Subnet Group và ENI |
| RDS bị chặn khi xóa | Chế độ bảo vệ xóa, tên snapshot cuối, bản sao lưu tự động được giữ lại và phê duyệt của người sở hữu |
| EBS vẫn phát sinh phí | Kiểm tra volume không gắn và các snapshot thực sự thuộc dự án |
| CloudWatch vẫn phát sinh phí | Thời gian lưu/lượng log thu nhận và alarm; xác nhận Agent đã dừng cùng EC2 |
| Chưa rõ tài nguyên thuộc về ai | Dừng xóa, dùng tag/danh mục tài nguyên và xin người sở hữu xác nhận |

Tiếp theo: [ghi nhận kết quả, thách thức và cải tiến tương lai](../5.11-Results-Challenges-Future/).
