---
title: "Tích hợp phần cứng"
date: "2026-07-28"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Tổng quan và mục tiêu

YOLO UNO là thiết bị chính của Workshop. Thiết bị đọc nhiệt độ, độ ẩm từ DHT20 và giá trị ánh sáng analog thô; điều khiển quạt, đèn/relay và servo rèm; gửi telemetry; định kỳ kiểm tra lệnh đang chờ; thực thi mỗi lệnh một lần rồi gửi ACK.

## Bước 1 - Nối phần cứng theo mã nguồn

File đang hoạt động `hardware/src/main.cpp` định nghĩa sơ đồ chân dưới đây. Các giá trị trong file này được ưu tiên hơn phần mô tả cũ ở nơi khác trong kho mã nguồn:

| Thành phần/tín hiệu | Định nghĩa firmware | Kết nối/hành vi trên YOLO UNO |
| :--- | :--- | :--- |
| Cảm biến ánh sáng analog | `LIGHT_SENSOR_PIN` | GPIO 1, Grove A1-A0/A0 |
| DHT20 và LCD1602 I2C | `I2C_SDA`, `I2C_SCL` | SDA GPIO 11, SCL GPIO 12 |
| Quạt | `PIN_FAN`, `PIN_FAN_CONTROL` | GPIO 10 và GPIO 17; GPIO 10 HIGH cùng GPIO 17 LOW làm quạt chạy |
| Đèn/relay | `PIN_LIGHT` | GPIO 6 |
| Servo rèm | `PIN_SERVO` | GPIO 38; đóng 0°, mở 90° |

Firmware tự dò LCD1602 tại các địa chỉ I2C `0x21`, `0x27`, `0x3F`. Không bổ sung cảm biến siêu âm hoặc cảm biến hiện diện. Dùng nguồn phù hợp cho thiết bị chấp hành và nối chung mass; không cấp dòng cho quạt hoặc servo trực tiếp từ GPIO.

## Bước 2 - Chuẩn bị PlatformIO

Mở dự án phần cứng trong VS Code và xác nhận:

- có board JSON YOLO UNO / ESP32-S3 và được tham chiếu đúng;
- `platformio.ini` chọn đúng môi trường và thư viện;
- baud Serial Monitor là `115200`;
- môi trường là `yolo_uno` trên ESP32-S3 với cấu hình bo mạch 8 MB theo mã nguồn;
- các thư viện phụ thuộc ArduinoJson, ESP32Servo, DHT20 và LiquidCrystal_I2C được tải thành công;
- `include/secrets.example.h` được commit; và
- `include/secrets.h` chỉ tồn tại cục bộ và đã được loại khỏi Git.

Dùng cấu trúc file bí mật sau:

```cpp
#pragma once
constexpr char WIFI_SSID[] = "<YOUR_WIFI_SSID>";
constexpr char WIFI_PASSWORD[] = "<YOUR_WIFI_PASSWORD>";
constexpr char API_BASE_URL[] = "http://<EC2_PUBLIC_IP>:8000";
constexpr char DEVICE_ID[] = "room_01";
```

Không công khai mật khẩu Wi-Fi thật. Nếu EC2 bị dừng rồi khởi động lại, hãy kiểm tra lại `API_BASE_URL`.

## Bước 3 - Xác minh cảm biến và thiết bị chấp hành tại chỗ

1. Khởi tạo I2C và xác nhận DHT20 phản hồi.
2. Đọc nhiệt độ, độ ẩm; loại bỏ giá trị không hợp lệ hoặc NaN.
3. Đọc giá trị ADC ánh sáng. Gọi đây là **giá trị ánh sáng analog**, không gọi là Lux cho đến khi mã nguồn có phép quy đổi đã hiệu chuẩn.
4. Khởi tạo đầu ra quạt và đèn ở trạng thái an toàn.
5. Gắn servo và kiểm tra vị trí đóng ở 0°, mở ở 90°.
6. Xác nhận LCD được tìm thấy ở một trong ba địa chỉ hỗ trợ.
7. Xác nhận lỗi cảm biến không làm bo mạch khởi động lại liên tục hoặc chặn vòng lặp xử lý lệnh.

Dùng mạch điều khiển và diode bảo vệ ngược cho tải cảm ứng khi cần. Không cấp dòng cho servo hoặc quạt trực tiếp từ GPIO.

## Bước 4 - Gửi telemetry

Firmware tuần tự hóa JSON bằng đúng các alias camelCase mà backend chấp nhận:

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

`lightIntensity` là giá trị analog thô, không phải Lux đã hiệu chuẩn. Mã nguồn gửi telemetry mỗi 5000 ms, thăm dò lệnh mỗi 2000 ms, cập nhật LCD mỗi 2000 ms, thử kết nối lại Wi-Fi mỗi 10000 ms với thời gian chờ kết nối 20000 ms và thời gian chờ HTTP 7000 ms.

```text
YOLO UNO đọc cảm biến → tạo JSON → POST /api/telemetry → kiểm tra trạng thái HTTP → chờ đến chu kỳ tiếp theo
```

Khi mã trạng thái báo lỗi, ghi lại phản hồi và thử lại sau một khoảng trễ có giới hạn. Không được để việc gửi mạng chặn logic an toàn của thiết bị chấp hành vô thời hạn.

## Bước 5 - Thăm dò, thực thi và xác nhận lệnh

Thăm dò lệnh:

```text
GET /api/devices/room_01/commands/latest
```

Firmware chấp nhận tám lệnh: `MODE_AUTO`, `MODE_MANUAL`, `FAN_ON`, `FAN_OFF`, `LIGHT_ON`, `LIGHT_OFF`, `CURTAIN_OPEN`, `CURTAIN_CLOSE`. Lệnh điều khiển trực tiếp sẽ chuyển firmware sang chế độ thủ công. Trong chế độ tự động, mã nguồn bật quạt khi nhiệt độ ≥30°C, bật đèn khi giá trị analog <350 và mở rèm khi giá trị analog <700.

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

Nếu gửi ACK thất bại sau khi thiết bị chấp hành đã hoạt động, chỉ thử gửi lại ACK, không thực thi lại thao tác. Mã nguồn lưu `autoMode`, `lastAck`, `pendingAck` trong ESP32 Preferences để có thể gửi lại ACK và tránh chạy lại cùng một lệnh sau khi bo mạch khởi động lại. Firmware từ chối lệnh không được hỗ trợ và không gửi ACK.

## Bước 6 - Biên dịch, nạp firmware và theo dõi

Trong PlatformIO terminal:

```bash
pio run
pio run --target upload
pio device monitor --baud 115200
```

Chuỗi thông báo mong đợi trên Serial Monitor; không sử dụng giá trị cảm biến cố định để giả lập kết quả:

```text
[wifi] connected
[telemetry] HTTP success for room_01
[command] pending command received: <COMMAND_ID> <SUPPORTED_COMMAND>
[actuator] command applied once
[ack] command acknowledged
```

## Kết quả mong đợi

YOLO UNO đọc DHT20 và cảm biến ánh sáng analog, cập nhật LCD1602, gửi đúng schema telemetry theo chu kỳ đã cấu hình, nhận mỗi lệnh được hỗ trợ một lần, điều khiển thiết bị chấp hành an toàn và chuyển lệnh tương ứng trên backend từ `Pending` sang `Executed` qua ACK. Bằng chứng Serial Monitor không được chứa mật khẩu Wi-Fi hoặc endpoint công khai chưa che.

<!-- TODO IMAGE: /images/5-Workshop/5.6-hardware/yolo-uno-hardware-setup.png — Toàn bộ mô hình YOLO UNO với DHT20, cảm biến ánh sáng analog, LCD1602, quạt, đèn/relay và servo rèm; không để lộ thông tin xác thực Wi-Fi. -->
<!-- TODO IMAGE: /images/5-Workshop/5.6-hardware/hardware-wiring-gpio-mapping.png — Cận cảnh wiring có chú thích GPIO 1, 6, 10, 11, 12, 17, 38 cùng nguồn an toàn/common ground. -->
<!-- TODO IMAGE: /images/5-Workshop/5.6-hardware/platformio-serial-monitor.png — PlatformIO Serial Monitor hiển thị Wi-Fi, telemetry, một lần thực thi lệnh và ACK; che địa chỉ IP và thông tin xác thực. -->

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| Không tìm thấy DHT20 | Chân SDA/SCL, địa chỉ, nguồn và quá trình khởi tạo I2C |
| Giá trị ánh sáng kẹt min/max | Khả năng ADC của pin, dải điện áp, dây |
| Bo mạch khởi động lại khi bật thiết bị chấp hành | Nguồn ngoài, dòng tiêu thụ, diode bảo vệ và nối chung mass |
| Wi-Fi lặp lại quá trình kết nối | SSID/mật khẩu, cường độ tín hiệu, khoảng trễ chặn và thời gian chờ giữa các lần thử |
| HTTP hết thời gian chờ | IP công khai, cổng 8000, EC2 SG, địa chỉ bind của Uvicorn và mạng máy khách |
| Lệnh bị lặp | So sánh ID lệnh và tách việc thực thi thiết bị khỏi việc gửi lại ACK |
| Lệnh giữ trạng thái `Pending` | URL/ID/nội dung ACK, phản hồi HTTP và log backend |
| Lệnh không được hỗ trợ | Ghi log và từ chối; không ACK thành `Executed` |

Tiếp theo: [kết nối React dashboard](../5.7-Frontend-Integration/).
