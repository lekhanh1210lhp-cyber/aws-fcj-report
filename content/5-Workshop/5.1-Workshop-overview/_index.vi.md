---
title: "Tổng quan Workshop"
date: "2026-07-28"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Tổng quan Workshop

## Bài toán và người dùng

Người vận hành cần một nơi duy nhất để theo dõi điều kiện môi trường và điều khiển thiết bị từ xa. Khi chưa có hệ thống tập trung, dữ liệu bị rời rạc, khó xem lịch sử và một lần bấm trên giao diện chưa chứng minh được thiết bị vật lý đã thực thi lệnh.

Workshop dành cho người quản lý phòng, người vận hành và người học muốn thực hành AWS, REST API, cơ sở dữ liệu, tích hợp frontend và IoT nhúng.

## Phạm vi giải pháp

Giải pháp giám sát `room_01` và hỗ trợ:

- telemetry nhiệt độ, độ ẩm và ánh sáng;
- dữ liệu mới nhất và lịch sử;
- điều khiển quạt, đèn và rèm;
- vòng đời command `Pending` → `Executed` có ACK từ thiết bị; và
- log backend, metric hạ tầng và alarm.

![Kiến trúc hệ thống cuối cùng](/images/2-Proposal/IoT_Dashboard_Architecture.png)

## Tiêu chí thành công

Workshop hoàn thành khi telemetry được lưu vào RDS và hiển thị trên dashboard, mọi command hỗ trợ đều truy vết được từ lúc tạo đến khi thiết bị thực thi và gửi ACK, đồng thời CloudWatch có đủ log và datapoint mong đợi.

**Kết quả mong đợi:** Phạm vi và tiêu chí nghiệm thu được xác định rõ trước khi tạo tài nguyên.

## Xử lý sự cố

- Nếu ảnh kiến trúc không hiển thị, kiểm tra file trong `static/images/2-Proposal/`.
- Nếu phát sinh yêu cầu mới, ghi vào hướng phát triển thay vì âm thầm thay đổi tiêu chí nghiệm thu.

Tiếp theo: [chuẩn bị tài khoản, công cụ và phần cứng](../5.2-Prerequisites/).
