---
title: "Kiểm thử và Xác thực End-to-End"
date: "2026-07-28"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

# Kiểm thử và Xác thực End-to-End

Chạy test theo thứ tự để khoanh vùng nguyên nhân khi có lỗi.

| ID | Test | Kết quả mong đợi | Bằng chứng |
|---|---|---|---|
| T01 | `GET /api/health` | HTTP 200 | curl hoặc Swagger |
| T02 | `POST /api/telemetry` | Có row mới trong RDS | response và SQL |
| T03 | Request latest/history | Trả dữ liệu `room_01` | JSON và chart |
| T04 | Tạo command | Trạng thái `Pending` | response và SQL |
| T05 | Device polling | Đúng actuator hoạt động một lần | Serial Monitor/video |
| T06 | Device ACK | Trạng thái thành `Executed` | SQL trước/sau |
| T07 | CloudWatch log/metric | Có event/datapoint mới | Ảnh Console |

## Trình tự xác thực

1. Lưu response health.
2. Gửi một mẫu telemetry biết trước và query row tương ứng trong PostgreSQL.
3. Xác nhận latest và history có cùng giá trị.
4. Tạo command hỗ trợ và chụp row `Pending`.
5. Bật hardware polling, quan sát thiết bị thực thi và lưu ACK.
6. Query lại cùng command và xác nhận `Executed`.

Kiểm thử thêm các trường hợp backend dừng, endpoint sai, mất Wi-Fi, lỗi kết nối RDS và command không hỗ trợ. UI và firmware phải hiển thị lỗi, không được báo đã thực thi.

**Kết quả mong đợi:** Bằng chứng API, database, dashboard, hardware và monitoring thể hiện một luồng thống nhất.

## Xử lý sự cố

- Dùng cùng một command ID trong request, database, Serial Monitor và bằng chứng ACK.
- Nếu test không ổn định, ghi timestamp và so sánh log backend, thiết bị và CloudWatch trước khi thử lại.

Tiếp theo: [cấu hình giám sát CloudWatch](../5.9-CloudWatch-Monitoring/).
