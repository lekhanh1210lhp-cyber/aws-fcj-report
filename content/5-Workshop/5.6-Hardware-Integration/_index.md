---
title: "Hardware Integration"
date: "2026-07-28"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Overview and objectives

YOLO UNO is the primary device for this workshop. It reads DHT20 temperature/humidity and a raw analog light value, controls fan, light/relay, and curtain servo outputs, sends telemetry, polls a pending command, executes it once, and sends ACK.

## Step 1 - Wire the source-defined hardware

The active `hardware/src/main.cpp` defines this map; it takes precedence over stale prose elsewhere in the source repository:

| Component/signal | Firmware definition | YOLO UNO connection/behavior |
| :--- | :--- | :--- |
| Analog light sensor | `LIGHT_SENSOR_PIN` | GPIO 1, Grove A1-A0/A0 |
| DHT20 and LCD1602 I2C | `I2C_SDA`, `I2C_SCL` | SDA GPIO 11, SCL GPIO 12 |
| Fan | `PIN_FAN`, `PIN_FAN_CONTROL` | GPIO 10 and GPIO 17; GPIO 10 HIGH with GPIO 17 LOW runs the fan |
| Light/relay | `PIN_LIGHT` | GPIO 6 |
| Curtain servo | `PIN_SERVO` | GPIO 38; close 0°, open 90° |

The firmware auto-detects LCD1602 I2C addresses `0x21`, `0x27`, and `0x3F`. Do not add ultrasonic or presence sensors. Use a suitable actuator supply and a common ground; never draw fan or servo current directly from a GPIO.

## Step 2 - Prepare PlatformIO

Open the hardware project in VS Code. Confirm:

- the YOLO UNO / ESP32-S3 board JSON exists and is referenced correctly;
- `platformio.ini` selects the intended environment and libraries;
- Serial Monitor baud is `115200`;
- the environment is `yolo_uno` on ESP32-S3 with the source-defined 8 MB board configuration;
- ArduinoJson, ESP32Servo, DHT20, and LiquidCrystal_I2C dependencies resolve;
- `include/secrets.example.h` is committed; and
- `include/secrets.h` is local and ignored.

Use this secret shape:

```cpp
#pragma once
constexpr char WIFI_SSID[] = "<YOUR_WIFI_SSID>";
constexpr char WIFI_PASSWORD[] = "<YOUR_WIFI_PASSWORD>";
constexpr char API_BASE_URL[] = "http://<EC2_PUBLIC_IP>:8000";
constexpr char DEVICE_ID[] = "room_01";
```

Never publish the real Wi-Fi password. If EC2 is stopped and started, re-check `API_BASE_URL`.

## Step 3 - Validate sensors and actuators locally

1. Initialize I2C and confirm the DHT20 responds.
2. Read temperature and humidity; reject invalid/NaN readings.
3. Read the light ADC value. Call it **analog light value**, not Lux, until the source contains a calibrated conversion.
4. Initialize fan and light outputs to a safe state.
5. Attach the servo and test close at 0° and open at 90°.
6. Confirm the LCD is detected at one of the three supported addresses.
7. Confirm a failed sensor does not repeatedly reset or block the command loop.

Use a driver and flyback protection for inductive loads where required. Servo/fan current must not be drawn directly from a GPIO.

## Step 4 - Send telemetry

The firmware serializes the exact camelCase aliases accepted by the backend:

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

`lightIntensity` is a raw analog value, not calibrated Lux. The source sends telemetry every 5000 ms, polls commands every 2000 ms, updates the LCD every 2000 ms, retries Wi-Fi every 10000 ms with a 20000 ms connection timeout, and uses a 7000 ms HTTP timeout.

```text
YOLO UNO read → JSON serialize → POST /api/telemetry → check HTTP status → wait configured interval
```

On non-success status, log the response and retry later with bounded delays. Do not block actuator safety logic indefinitely.

## Step 5 - Poll, execute, and acknowledge commands

Poll:

```text
GET /api/devices/room_01/commands/latest
```

Accept the eight firmware commands: `MODE_AUTO`, `MODE_MANUAL`, `FAN_ON`, `FAN_OFF`, `LIGHT_ON`, `LIGHT_OFF`, `CURTAIN_OPEN`, and `CURTAIN_CLOSE`. A direct actuator command switches the firmware to manual mode. In auto mode, the source turns the fan on at temperature ≥30°C, turns the light on when the analog value is <350, and opens the curtain when the analog value is <700.

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

If ACK fails after the actuator succeeds, retry the ACK without applying the actuator again. The source stores `autoMode`, `lastAck`, and `pendingAck` in ESP32 Preferences so an ACK can be retried and the same command is not re-executed after reboot. Unsupported commands are rejected by firmware and are not acknowledged.

## Step 6 - Build, upload, and monitor

In a PlatformIO terminal:

```bash
pio run
pio run --target upload
pio device monitor --baud 115200
```

Expected Serial Monitor sequence, without fabricated fixed sensor values:

```text
[wifi] connected
[telemetry] HTTP success for room_01
[command] pending command received: <COMMAND_ID> <SUPPORTED_COMMAND>
[actuator] command applied once
[ack] command acknowledged
```

## Expected Result

YOLO UNO reads DHT20 and the analog light sensor, updates LCD1602, posts the exact telemetry schema every configured interval, receives each supported command once, applies safe actuator state, and changes the matching backend command from `Pending` to `Executed` through ACK. Serial evidence contains no Wi-Fi password or unredacted public endpoint.

<!-- TODO IMAGE: /images/5-Workshop/5.6-hardware/yolo-uno-hardware-setup.png — Full YOLO UNO setup with DHT20, analog light sensor, LCD1602, fan, light/relay, and curtain servo; do not expose Wi-Fi credentials. -->
<!-- TODO IMAGE: /images/5-Workshop/5.6-hardware/hardware-wiring-gpio-mapping.png — Annotated wiring close-up showing GPIO 1, 6, 10, 11, 12, 17, and 38 plus safe power/common ground. -->
<!-- TODO IMAGE: /images/5-Workshop/5.6-hardware/platformio-serial-monitor.png — PlatformIO Serial Monitor showing Wi-Fi, telemetry, one command execution, and ACK; redact IPs and credentials. -->

## Troubleshooting

| Symptom | Check |
| :--- | :--- |
| DHT20 not found | SDA/SCL, address, power, I2C initialization |
| Light value stuck at min/max | ADC pin capability, voltage range, wiring |
| Board resets on actuator action | External supply, current, flyback protection, common ground |
| Wi-Fi reconnect loop | SSID/password, signal, blocking delay, reconnect backoff |
| HTTP timeout | Public IP, port 8000, EC2 SG, Uvicorn bind, client network |
| Command repeats | Compare command IDs and separate actuator execution from ACK retry |
| Command stays `Pending` | ACK URL/ID/body, HTTP response, backend log |
| Unsupported command | Log and reject it; do not ACK it as executed |

Next: [connect the React dashboard](../5.7-Frontend-Integration/).
