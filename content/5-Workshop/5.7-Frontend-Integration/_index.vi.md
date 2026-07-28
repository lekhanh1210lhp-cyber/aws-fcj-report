---
title: "Tích hợp Frontend"
date: "2026-07-28"
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# Tích hợp Frontend

## Bước 1 - Cấu hình Vite

Cài và chạy frontend:

```bash
cd frontend
npm install
npm run dev
```

Cấu hình Vite development proxy để `/api` trỏ tới `http://<EC2_PUBLIC_IP>:8000`. Component nên gọi relative path như `/api/telemetry/latest` thay vì nhúng địa chỉ EC2 ở nhiều nơi trong code.

## Bước 2 - Liên kết dữ liệu và control

Triển khai:

- dữ liệu mới nhất và biểu đồ lịch sử cho `room_01`;
- điều khiển thủ công quạt, đèn và rèm;
- trạng thái command dựa trên dữ liệu backend, không dùng local toggle lạc quan; và
- chế độ manual và automatic/recommendation.

Nếu gợi ý tự động chỉ dựa trên ngưỡng, gọi đúng là **rule-based recommendation**, không gọi là machine learning.

## Bước 3 - Kiểm tra browser traffic

Dùng DevTools **Network** để kiểm tra request URL, HTTP status, JSON response và chu kỳ refresh. Khi bấm control, hệ thống phải tạo command và hiển thị trạng thái do server trả về.

**Kết quả mong đợi:** Telemetry hiển thị được, history cập nhật và mỗi control tạo một command `Pending` có thể truy vết.

## Xử lý sự cố

- Vite 404 thường do proxy path sai; CORS error cho thấy request đang bỏ qua proxy hoặc policy backend chưa đủ.
- Nếu UI báo thành công trước khi hardware thực thi, liên kết giao diện với command status và ACK thay vì state local của component.

Tiếp theo: [xác thực toàn bộ hệ thống](../5.8-End-to-End-Testing/).
