---
title: "Chi phí, bảo mật và dọn dẹp"
date: "2026-07-28"
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

## Tổng quan và mục tiêu

Xác định các yếu tố phát sinh chi phí, rà soát ranh giới bảo mật của mô hình thử nghiệm, lưu lại bằng chứng cần thiết và xóa tài nguyên theo đúng thứ tự phụ thuộc. Không nêu mức giá cố định vì chi phí phụ thuộc vào khu vực, kích thước tài nguyên, thời gian lưu trữ, lưu lượng truyền dữ liệu và bảng giá tại thời điểm sử dụng.

## Bước 1 - Rà soát các yếu tố tạo chi phí

| Tài nguyên | Yếu tố phát sinh chi phí | Cách kiểm soát trong Workshop |
| :--- | :--- | :--- |
| Amazon EC2 | Loại instance và số giờ chạy | Dùng instance nhỏ đã duyệt; dừng hoặc chấm dứt sau khi sử dụng |
| Amazon EBS | Loại ổ đĩa, dung lượng cấp phát và snapshot | Chọn dung lượng phù hợp; xóa volume/snapshot không còn gắn |
| Amazon RDS for PostgreSQL | DB class, thời gian chạy, lưu trữ và sao lưu | Dùng cơ sở dữ liệu nhỏ; xóa sau khi thu đủ bằng chứng nếu được duyệt |
| Amazon CloudWatch | Metric tùy chỉnh, lượng log thu nhận/lưu trữ và alarm | Chỉ thu metric hệ điều hành mỗi 60 giây khi cần; đặt thời gian lưu ngắn |
| Truyền dữ liệu | Lưu lượng giữa người dùng/thiết bị và EC2 | Chọn chu kỳ gửi telemetry và thăm dò hợp lý |

Dùng AWS Pricing Calculator hoặc hóa đơn thực tế để lập bản ước tính có ghi rõ ngày. Không đưa mức giá chưa được kiểm chứng vào báo cáo.

## Bước 2 - Rà soát ranh giới bảo mật

- Dùng danh tính theo nguyên tắc đặc quyền tối thiểu và bật MFA.
- Dùng EC2 IAM Role; không ghi cố định AWS access key trong mã nguồn.
- Giới hạn SSH port 22 theo `<ADMIN_IP>/32`.
- Chỉ cho RDS 5432 nhận từ EC2 Security Group.
- Giữ RDS private.
- Loại `.env`, `.pem`, `.key` và `hardware/include/secrets.h` khỏi Git.
- Dùng placeholder trong tài liệu và che thông tin nhạy cảm trên ảnh.
- Xem việc mở công khai cổng HTTP 8000 là một hạn chế của bản demo.
- Rà soát quy tắc đi ra, thời gian lưu log, người dùng cơ sở dữ liệu và tag tài nguyên.
- Thay mới ngay mọi thông tin bí mật nếu chúng xuất hiện trong Git, lịch sử terminal, ảnh hoặc video demo.

Trước khi đưa hệ thống vào vận hành thực tế, cần bổ sung reverse proxy, HTTPS, xác thực, phân quyền, quản lý bí mật, sao lưu và một thiết kế mạng đã được rà soát. Các thành phần này chưa được triển khai trong phiên bản hiện tại.

## Bước 3 - Lưu bằng chứng trước khi dọn dẹp

Trước khi xóa, lưu:

1. sơ đồ kiến trúc và danh mục tài nguyên;
2. trạng thái EC2/dịch vụ và mã commit đã triển khai;
3. bằng chứng bảng/câu truy vấn RDS không chứa thông tin xác thực;
4. kết quả kiểm thử telemetry, lệnh và ACK;
5. ảnh CloudWatch log/metric/alarm; và
6. bằng chứng demo phần cứng/frontend.

Xác nhận người sở hữu snapshot và thời gian phải lưu bằng chứng.

## Danh sách tài nguyên AWS trước khi dọn dẹp

Bảng dưới đây tổng hợp các tài nguyên dự án đang sử dụng. Trước khi dừng hoặc xóa bất kỳ tài nguyên nào, nhóm cần lưu đầy đủ ảnh chụp màn hình, log, kết quả kiểm thử, mã nguồn và video minh họa.

| Tài nguyên | Tên hoặc vai trò | Trạng thái hiện tại | Bằng chứng | Hành động dọn dẹp |
| :--- | :--- | :--- | :--- | :--- |
| Amazon EC2 | `iot-backend-server` | Đang chạy, loại `t3.micro`, vượt qua `3/3` kiểm tra trạng thái | Trang EC2 Instances | Lưu mã nguồn, cấu hình và log cần thiết; sau đó dừng hoặc chấm dứt instance |
| Amazon EBS | Volume gốc của `iot-backend-server` | `gp3`, 10 GiB, trạng thái `In-use`, `Okay`, `I/O Normal`, chưa mã hóa | Trang EC2 → Volumes | Kiểm tra `Delete on termination`; tạo snapshot nếu cần, sau đó xóa volume khi không còn sử dụng |
| Amazon RDS for PostgreSQL | `iot-dashboard-db` | Trạng thái `Available`, PostgreSQL, loại `db.t4g.micro`, Internet access gateway ở trạng thái `Disabled` | Trang RDS Database | Tạo snapshot cuối cùng nếu cần, sau đó xóa DB instance |
| DB Subnet Group | `rds-ec2-db-subnet-group-1` | Đang được RDS sử dụng | Phần RDS Connectivity | Chỉ xóa sau khi DB instance và các tài nguyên phụ thuộc đã được xóa |
| EC2 Security Groups | `iot-backend-sg`, `ec2-rds-1` | Đang được sử dụng để kiểm soát SSH, cổng backend và kết nối đến RDS | Tab EC2 Security | Xóa sau khi EC2 và các network interface liên quan không còn sử dụng |
| RDS Security Group | `rds-ec2-1` | In use; cho phép kết nối từ EC2 Security Group | Phần RDS Security Group rules | Xóa sau khi RDS và các network interface phụ thuộc đã được xóa |
| IAM Role | `iot-dashboard-cloudwatch-role` | Đang được gắn với EC2 | EC2 Security và trang IAM Role | Tháo role khỏi EC2, kiểm tra tài nguyên phụ thuộc, sau đó xóa role |
| IAM Policy | `CloudWatchAgentServerPolicy` | AWS-managed policy đang gắn với IAM Role | IAM Permissions | Không xóa AWS-managed policy; chỉ tháo hoặc xóa IAM Role của dự án |
| CloudWatch Log Group | `/aws/ec2/aws-iot-dashboard/backend` | Active, chứa FastAPI access logs và HTTP status | CloudWatch Logs | Lưu log cần thiết; sau đó cấu hình retention hoặc xóa log group |
| CloudWatch Dashboard | `ec2-rds-metrics` | Trạng thái `Active`; hiển thị metric EC2 và RDS | CloudWatch Dashboard | Lưu ảnh metric, sau đó xóa dashboard khi không còn cần thiết |
| CloudWatch Alarms | 5 alarm cho EC2 và RDS | Một số ở trạng thái `OK`, một số `Insufficient data`; chưa gắn action thông báo | CloudWatch Alarms | Lưu bằng chứng cấu hình, sau đó xóa toàn bộ alarm |
| Dịch vụ FastAPI backend | `aws-iot-backend` trên EC2 | Đang cung cấp REST API và ghi log truy cập | EC2 và CloudWatch Logs | Sao lưu mã nguồn, service file và cấu hình môi trường trước khi chấm dứt EC2 |
| Tệp firmware | `firmware.bin` | Được build thành công bằng PlatformIO trong môi trường `yolo_uno` | Terminal hiển thị `SUCCESS` | Lưu trên máy cục bộ hoặc trong kho artifact; đây không phải tài nguyên AWS cần xóa |

> **Lưu ý:** Không dọn dẹp tài nguyên trước khi lưu đầy đủ ảnh chụp màn hình, log, kết quả kiểm thử, mã nguồn và video minh họa cần thiết. Volume EBS hiện tại chưa được mã hóa; khi triển khai production, nên bật EBS encryption bằng AWS KMS.

<!-- TODO: Capture this rendered table as aws-resource-inventory.png -->

## Bước 4 - Chỉ dọn dẹp tài nguyên thuộc dự án

1. Lưu ảnh chụp màn hình, log, mã nguồn, kết quả kiểm thử và video minh họa; dừng telemetry/lệnh mới và đưa thiết bị chấp hành về trạng thái an toàn.
2. Sao lưu mã nguồn backend, service file và cấu hình môi trường cần thiết.
3. Quyết định có cần giữ dữ liệu bằng snapshot RDS cuối cùng đã được phê duyệt hay không; tạo snapshot nếu cần.
4. Dừng hoặc chấm dứt `iot-backend-server` sau khi được phê duyệt bàn giao.
5. Kiểm tra `DeleteOnTermination` của volume EBS gốc; chỉ xóa volume hoặc snapshot thuộc dự án khi không còn cần thiết.
6. Xóa `iot-dashboard-db` khi không còn cần cơ sở dữ liệu và dữ liệu phải lưu.
7. Xóa năm CloudWatch Alarm của dự án đã ghi trong mục 5.9.
8. Xóa CloudWatch Dashboard `ec2-rds-metrics` sau khi lưu bằng chứng.
9. Cấu hình retention hoặc xóa `/aws/ec2/aws-iot-dashboard/backend` sau khi lưu log cần thiết.
10. Tháo và xóa `iot-dashboard-cloudwatch-role` cùng instance profile chỉ khi không còn workload nào khác sử dụng. Không xóa AWS-managed policy `CloudWatchAgentServerPolicy`.
11. Chỉ xóa `iot-backend-sg`, `ec2-rds-1` và `rds-ec2-1` sau khi không còn EC2, RDS, ENI hoặc Security Group phụ thuộc.
12. Chỉ xóa `rds-ec2-db-subnet-group-1` khi RDS không còn sử dụng. Không xóa tài nguyên VPC được chọn sẵn hoặc dùng chung.
13. Kiểm tra lại EC2, EBS, RDS, CloudWatch, Billing/Cost Explorer và danh mục tài nguyên theo tag để bảo đảm không còn tài nguyên nào của dự án ngoài dự kiến.

Cách triển khai hiện tại không tạo S3 bucket và không có stack hoặc state của CloudFormation, SAM, CDK hay Terraform. Vì vậy, quy trình dọn dẹp **không** yêu cầu xóa bucket hoặc stack. Nếu phiên bản sau bổ sung các tài nguyên này, cần cập nhật danh mục và thứ tự phụ thuộc trước.

Dừng RDS chỉ là biện pháp tạm thời và chịu giới hạn của dịch vụ; RDS có thể tự khởi động lại. Muốn tránh chi phí dài hạn, cần xóa cơ sở dữ liệu và các tài nguyên tính phí khác theo quyết định lưu giữ của nhóm.

## Bước 5 - Xác minh kết quả dọn dẹp

- Kiểm kê lại tài nguyên theo tag trong đúng khu vực.
- Kiểm tra instance đã dừng, volume EBS không còn gắn, snapshot RDS được giữ lại, log group và alarm không còn sử dụng.
- Nếu không xóa được Security Group, hãy tìm ENI hoặc tài nguyên phụ thuộc thay vì cưỡng chế xóa.
- Nếu không xóa được cơ sở dữ liệu, rà soát chế độ bảo vệ xóa và yêu cầu snapshot với người sở hữu.
- Ghi ID của từng tài nguyên đã xóa trong bằng chứng dọn dẹp; không để lộ thông tin xác thực.

## Kết quả mong đợi

Các bằng chứng cần thiết được lưu lại; không còn tài nguyên có tính phí của dự án nằm ngoài phạm vi được phê duyệt; tài nguyên dùng chung không bị ảnh hưởng; phần rà soát bảo mật phản ánh đúng các hạn chế hiện tại và không tuyên bố hệ thống đã sẵn sàng để vận hành thực tế.

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| Không xóa được Security Group | Tìm ENI, EC2, RDS hoặc Security Group tham chiếu còn phụ thuộc |
| Không xóa được VPC/subnet | Kiểm tra Internet Gateway, route table association, DB Subnet Group và ENI |
| RDS bị chặn khi xóa | Chế độ bảo vệ xóa, tên snapshot cuối, bản sao lưu tự động được giữ lại và phê duyệt của người sở hữu |
| EBS vẫn phát sinh phí | Kiểm tra volume không gắn và các snapshot thực sự thuộc dự án |
| CloudWatch vẫn phát sinh phí | Thời gian lưu/lượng log thu nhận và alarm; xác nhận Agent đã dừng cùng EC2 |
| Chưa rõ tài nguyên thuộc về ai | Dừng xóa, dùng tag/danh mục tài nguyên và xin người sở hữu xác nhận |

Tiếp theo: [ghi nhận kết quả, thách thức và cải tiến tương lai](../5.11-results-challenges-future/).
