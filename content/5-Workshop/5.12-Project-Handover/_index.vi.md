---
title: "Bàn giao Dự án"
date: "2026-07-28"
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

# Bàn giao Dự án

## Sản phẩm bàn giao

Cung cấp:

- URL source repository và cây thư mục rõ ràng;
- `README.md` và `README.vi.md`;
- hướng dẫn chạy backend, frontend và hardware;
- `.env.example` và `secrets.example.h` chỉ có placeholder;
- hướng dẫn migration hoặc khởi tạo database;
- bằng chứng test từ Mục 5.8;
- bằng chứng CloudWatch và trạng thái dọn dẹp; và
- link video demo cùng tóm tắt đóng góp của thành viên.

## Checklist vận hành

1. Cấu hình secret database, Wi-Fi, device ID và API endpoint.
2. Xác nhận cấu hình EC2, RDS, IAM Role và Security Group.
3. Khởi động `aws-iot-backend` và kiểm tra `/api/health`.
4. Khởi động frontend và xác nhận cấu hình Vite proxy.
5. Cấp nguồn hardware và xem Serial Monitor.
6. Chạy smoke test telemetry và command/ACK.
7. Kiểm tra CloudWatch log, metric và alarm.

## Review cuối

- [ ] Cả 12 mục có trang tiếng Anh và tiếng Việt tương ứng.
- [ ] Command, endpoint, service name và screenshot nhất quán.
- [ ] Không còn placeholder trong cấu hình được tuyên bố hoạt động.
- [ ] Không commit secret hoặc private key.
- [ ] Hugo site build thành công và navigation hoạt động.
- [ ] Mỗi mục kỹ thuật được ít nhất một thành viên khác review.

**Kết quả mong đợi:** Người vận hành mới có thể cấu hình, chạy, xác thực, giám sát và dọn dẹp project an toàn mà không phụ thuộc kiến thức chưa được ghi lại của nhóm.

Workshop AWS IoT Dashboard kết thúc tại đây.
