---
title: "Tích hợp phần cứng"
date: "2026-07-28"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Tổng quan và mục tiêu

YOLO UNO là thiết bị chính của Workshop. Thiết bị đọc nhiệt độ/độ ẩm từ DHT20 và giá trị ánh sáng analog thô, điều khiển quạt, đèn/relay và servo rèm, gửi telemetry, polling command đang pending, thực thi một lần rồi gửi ACK.

## Bước 1 - Nối phần cứng theo source

File đang hoạt động `hardware/src/main.cpp` định nghĩa pin map dưới đây; giá trị này được ưu tiên hơn phần mô tả cũ ở nơi khác trong source repository:

| Thành phần/tín hiệu | Định nghĩa firmware | Kết nối/hành vi trên YOLO UNO |
| :--- | :--- | :--- |
| Cảm biến ánh sáng analog | `LIGHT_SENSOR_PIN` | GPIO 1, Grove A1-A0/A0 |
| DHT20 và LCD1602 I2C | `I2C_SDA`, `I2C_SCL` | SDA GPIO 11, SCL GPIO 12 |
| Quạt | `PIN_FAN`, `PIN_FAN_CONTROL` | GPIO 10 và GPIO 17; GPIO 10 HIGH cùng GPIO 17 LOW làm quạt chạy |
| Đèn/relay | `PIN_LIGHT` | GPIO 6 |
| Servo rèm | `PIN_SERVO` | GPIO 38; đóng 0°, mở 90° |

Firmware tự dò LCD1602 tại địa chỉ I2C `0x21`, `0x27`, `0x3F`. Không thêm ultrasonic hoặc presence sensor. Dùng nguồn actuator phù hợp và common ground; không lấy dòng quạt hoặc servo trực tiếp từ GPIO.

## Bước 2 - Chuẩn bị PlatformIO

Mở hardware project trong VS Code. Xác nhận:

- có board JSON YOLO UNO / ESP32-S3 và được tham chiếu đúng;
- `platformio.ini` chọn đúng environment và library;
- baud Serial Monitor là `115200`;
- environment là `yolo_uno` trên ESP32-S3 với cấu hình board 8 MB theo source;
- các dependency ArduinoJson, ESP32Servo, DHT20 và LiquidCrystal_I2C được resolve;
- `include/secrets.example.h` được commit; và
- `include/secrets.h` chỉ ở local và đã ignore.

Dùng cấu trúc secret sau:

```cpp
#pragma once
constexpr char WIFI_SSID[] = "<YOUR_WIFI_SSID>";
constexpr char WIFI_PASSWORD[] = "<YOUR_WIFI_PASSWORD>";
constexpr char API_BASE_URL[] = "http://<EC2_PUBLIC_IP>:8000";
constexpr char DEVICE_ID[] = "room_01";
```

Không công khai Wi-Fi password thật. Nếu EC2 stop rồi start, kiểm tra lại `API_BASE_URL`.

## Bước 3 - Xác minh cảm biến và actuator tại chỗ

1. Khởi tạo I2C và xác nhận DHT20 phản hồi.
2. Đọc nhiệt độ, độ ẩm; loại giá trị invalid/NaN.
3. Đọc ADC ánh sáng. Gọi là **giá trị ánh sáng analog**, không gọi Lux đến khi source có phép đổi đã hiệu chuẩn.
4. Khởi tạo output quạt, đèn ở trạng thái an toàn.
5. Attach servo và test đóng ở 0°, mở ở 90°.
6. Xác nhận LCD được tìm thấy ở một trong ba địa chỉ hỗ trợ.
7. Xác nhận cảm biến lỗi không liên tục reset hoặc chặn command loop.

Dùng driver và flyback protection cho tải cảm ứng khi cần. Không lấy dòng servo/quạt trực tiếp từ GPIO.

## Bước 4 - Gửi telemetry

Firmware serialize đúng alias camelCase mà backend chấp nhận:

```json
{
  "deviceId": "room_01",
  "temperature": 25.0,
  "humidity": 60.0,
  "lightIntensity": 512,
  "fan": false,
  "light": false,
  "curtain": false
}
```

`lightIntensity` là giá trị analog thô, không phải Lux đã hiệu chuẩn. Source gửi telemetry mỗi 5000 ms, polling command mỗi 2000 ms, cập nhật LCD mỗi 2000 ms, retry Wi-Fi mỗi 10000 ms với timeout kết nối 20000 ms và dùng HTTP timeout 7000 ms.

```text
YOLO UNO đọc → serialize JSON → POST /api/telemetry → kiểm tra HTTP status → chờ interval đã cấu hình
```

Khi status không thành công, log response và retry sau với delay có giới hạn. Không chặn logic an toàn actuator vô thời hạn.

## Bước 5 - Polling, thực thi và ACK command

Polling:

```text
GET /api/devices/room_01/commands/latest
```

Nhận tám command firmware: `MODE_AUTO`, `MODE_MANUAL`, `FAN_ON`, `FAN_OFF`, `LIGHT_ON`, `LIGHT_OFF`, `CURTAIN_OPEN`, `CURTAIN_CLOSE`. Command actuator trực tiếp chuyển firmware sang manual mode. Trong auto mode, source bật quạt khi nhiệt độ ≥30°C, bật đèn khi giá trị analog <350 và mở rèm khi giá trị analog <700.

```cpp
if (pending && commandId != lastExecutedCommandId) {
  const bool applied = applySupportedCommand(command);
  if (applied) {
    lastExecutedCommandId = commandId;
    sendAck(commandId);
  }
}
```

ACK:

```text
POST /api/devices/room_01/commands/{command_id}/ack
```

Nếu ACK lỗi sau khi actuator thành công, retry ACK mà không áp dụng actuator lần nữa. Source lưu `autoMode`, `lastAck`, `pendingAck` trong ESP32 Preferences để retry ACK và tránh thực thi lại cùng command sau reboot. Firmware từ chối command không hỗ trợ và không gửi ACK.

## Bước 6 - Build, upload và monitor

Trong PlatformIO terminal:

```bash
pio run
pio run --target upload
pio device monitor --baud 115200
```

Chuỗi Serial Monitor mong đợi, không bịa giá trị cảm biến cố định:

```text
[wifi] connected
[telemetry] HTTP success for room_01
[command] pending command received: <COMMAND_ID> <SUPPORTED_COMMAND>
[actuator] command applied once
[ack] command acknowledged
```

<!-- TODO IMAGE: /images/5-Workshop/5.6-hardware/yolo-uno-hardware-setup.png — Toàn bộ setup YOLO UNO với DHT20, cảm biến ánh sáng analog, LCD1602, quạt, đèn/relay và servo rèm; không để lộ credential Wi-Fi. -->
<!-- TODO IMAGE: /images/5-Workshop/5.6-hardware/hardware-wiring-gpio-mapping.png — Cận cảnh wiring có chú thích GPIO 1, 6, 10, 11, 12, 17, 38 cùng nguồn an toàn/common ground. -->
<!-- TODO IMAGE: /images/5-Workshop/5.6-hardware/platformio-serial-monitor.png — PlatformIO Serial Monitor hiển thị Wi-Fi, telemetry, một lần thực thi command và ACK; che IP và credential. -->

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| Không tìm thấy DHT20 | SDA/SCL, address, power, I2C initialization |
| Giá trị ánh sáng kẹt min/max | Khả năng ADC của pin, dải điện áp, dây |
| Board reset khi bật actuator | Nguồn ngoài, dòng, flyback protection, common ground |
| Wi-Fi reconnect loop | SSID/password, signal, blocking delay, reconnect backoff |
| HTTP timeout | Public IP, port 8000, EC2 SG, Uvicorn bind, mạng client |
| Command lặp | So command ID và tách actuator execution khỏi ACK retry |
| Command giữ `Pending` | ACK URL/ID/body, HTTP response, backend log |
| Command không hỗ trợ | Log và từ chối; không ACK thành executed |

Tiếp theo: [kết nối React dashboard](../5.7-Frontend-Integration/).
