---
title: "Tích hợp Phần cứng"
date: "2026-07-28"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Tích hợp Phần cứng

## Bước 1 - Đấu nối và cấu hình

Ghi lại pin mapping chính xác của project trước khi cấp nguồn. Kết nối DHT20 và cảm biến ánh sáng, sau đó kết nối quạt, đèn/relay và servo rèm. Kiểm tra yêu cầu điện áp, dòng điện; không cấp tải dòng cao trực tiếp từ chân microcontroller.

Tạo `include/secrets.h` đã được Git ignore:

```cpp
#pragma once
#define WIFI_SSID "<WIFI_SSID>"
#define WIFI_PASSWORD "<WIFI_PASSWORD>"
#define API_BASE_URL "http://<EC2_PUBLIC_IP>:8000"
#define DEVICE_ID "room_01"
```

Commit `secrets.example.h` chỉ có placeholder, không commit file thật.

## Bước 2 - Telemetry và command

Firmware cần:

1. kết nối Wi-Fi;
2. đọc cảm biến;
3. `POST /api/telemetry`;
4. polling command mới nhất cho `room_01`;
5. thực thi một trong sáu thao tác quạt/đèn/rèm được hỗ trợ;
6. chỉ gửi ACK sau khi thực thi thành công; và
7. ghi nhớ command ID để tránh thực thi lặp.

Build và upload:

```bash
pio run
pio run --target upload
pio device monitor
```

**Kết quả mong đợi:** Serial Monitor hiển thị request telemetry thành công, command chỉ được thực thi một lần và ACK chuyển trạng thái thành `Executed`.

## Xử lý sự cố

- Nếu Wi-Fi kết nối nhưng HTTP lỗi, kiểm tra `API_BASE_URL`, thay đổi EC2 public IP, cổng `8000` và Security Group.
- Nếu actuator làm board reset, dùng nguồn ngoài phù hợp và nối common ground.
- Không ACK command không hỗ trợ hoặc thực thi thất bại; thay vào đó ghi log nguyên nhân.

Tiếp theo: [kết nối React dashboard](../5.7-Frontend-Integration/).
