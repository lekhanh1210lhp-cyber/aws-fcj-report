---
title: "Chi phí, bảo mật và dọn dẹp"
date: "2026-07-28"
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

## Tổng quan và mục tiêu

Hiểu cost driver, rà soát ranh giới bảo mật của prototype, lưu bằng chứng cần thiết và xóa tài nguyên theo thứ tự phụ thuộc. Không nêu giá chính xác vì chi phí phụ thuộc region, kích thước tài nguyên, retention, data transfer và bảng giá hiện hành.

## Rà soát chi phí

| Tài nguyên | Cost driver | Kiểm soát trong Workshop |
| :--- | :--- | :--- |
| Amazon EC2 | Instance type và giờ chạy | Instance nhỏ đã duyệt; stop/terminate sau khi dùng |
| Amazon EBS | Loại/dung lượng cấp phát và snapshot | Right-size, xóa volume/snapshot không còn gắn |
| Amazon RDS for PostgreSQL | DB class, thời gian chạy, storage, backup | DB nhỏ; xóa sau evidence nếu được duyệt |
| Amazon CloudWatch | Custom metric, log ingest/storage, alarm | Guest metric 60 giây khi cần; retention ngắn |
| Data transfer | Traffic giữa user/device và EC2 | Interval telemetry và polling hợp lý |

Dùng AWS Pricing Calculator hoặc bill thật để có estimate gắn ngày. Không chép giá chưa xác minh vào báo cáo.

## Rà soát bảo mật

- Dùng identity least privilege và MFA.
- Dùng EC2 IAM Role; không hard-code AWS access key.
- Giới hạn SSH port 22 theo `<ADMIN_IP>/32`.
- Chỉ cho RDS 5432 nhận từ EC2 Security Group.
- Giữ RDS private.
- Ignore `.env`, `.pem`, `.key` và `hardware/include/secrets.h`.
- Dùng placeholder trong tài liệu và che thông tin nhạy cảm trên ảnh.
- Xem HTTP public port 8000 là hạn chế demo.
- Rà soát outbound rule, log retention, database user và resource tag.
- Rotate ngay mọi secret nếu xuất hiện trong Git, terminal history, ảnh hoặc demo video.

Khuyến nghị production gồm reverse proxy, HTTPS, authentication, authorization, managed secrets, backup và thiết kế mạng đã review. Đây chưa phải tính năng triển khai hiện tại.

## Bằng chứng trước khi dọn dẹp

Trước khi xóa, lưu:

1. kiến trúc và resource inventory;
2. EC2/service health và deployment commit;
3. bằng chứng table/query RDS không có credential;
4. bản ghi test telemetry, command và ACK;
5. ảnh CloudWatch log/metric/alarm; và
6. bằng chứng demo phần cứng/frontend.

Xác nhận người sở hữu snapshot và thời gian phải giữ evidence.

## Quy trình dọn dẹp

1. Dừng traffic telemetry/command mới và đưa actuator về trạng thái an toàn.
2. Stop hoặc terminate EC2 theo quyết định bàn giao.
3. Xóa EBS volume không gắn và snapshot không cần sau khi xác nhận owner.
4. Xóa RDS, chỉ tạo final snapshot khi việc lưu giữ được yêu cầu và duyệt.
5. Xóa CloudWatch alarm.
6. Xóa log group và dữ liệu giám sát không cần theo retention policy.
7. Xóa IAM Role/instance profile nếu chỉ dùng cho project.
8. Chỉ xóa Security Group sau khi ENI/resource phụ thuộc không còn.
9. Chỉ xóa route/network resource của Workshop khi không có dependency dùng chung.
10. Mở Billing/Cost Explorer và xác minh không còn tài nguyên project ngoài dự kiến.

Stop RDS chỉ là tạm thời và chịu giới hạn dịch vụ; RDS có thể tự khởi động lại. Xóa database và tài nguyên tính phí khác mới tránh chi phí dài hạn, tùy quyết định retention của nhóm.

## Xác minh và xử lý sự cố

- Chạy lại inventory theo tag trong đúng region.
- Kiểm tra instance stopped, EBS volume không gắn, RDS snapshot giữ lại, log group và alarm nhàn rỗi.
- Nếu không xóa được Security Group, tìm ENI/resource phụ thuộc thay vì force.
- Nếu không xóa được database, rà soát deletion protection và yêu cầu snapshot với owner.
- Ghi resource ID đã xóa trong evidence dọn dẹp; không để lộ credential.

<!-- TODO IMAGE: /images/5-Workshop/5.10-cleanup/aws-resource-inventory.png — Resource inventory AWS trước/sau và kiểm tra Billing/Cost Explorer đã che thông tin nhạy cảm, thể hiện kết quả dọn dẹp Workshop. -->

**Kết quả mong đợi:** giữ đủ evidence, không còn tài nguyên Workshop tính phí ngoài phê duyệt và security review ghi đúng hạn chế hiện tại mà không tuyên bố production-ready.

Tiếp theo: [ghi nhận kết quả, thách thức và cải tiến tương lai](../5.11-Results-Challenges-Future/).
