---
title: "Hardware Integration"
date: "2026-07-28"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Hardware Integration

## Step 1 - Wire and configure

Record the exact project pin map before powering the circuit. Connect the DHT20 and light sensor, then the fan, light/relay, and curtain servo. Confirm voltage and current requirements; do not drive a high-current load directly from a microcontroller pin.

Create an ignored `include/secrets.h`:

```cpp
#pragma once
#define WIFI_SSID "<WIFI_SSID>"
#define WIFI_PASSWORD "<WIFI_PASSWORD>"
#define API_BASE_URL "http://<EC2_PUBLIC_IP>:8000"
#define DEVICE_ID "room_01"
```

Commit a `secrets.example.h` containing placeholders, never the real file.

## Step 2 - Telemetry and commands

The firmware should:

1. connect to Wi-Fi;
2. sample the sensors;
3. `POST /api/telemetry`;
4. poll the latest command for `room_01`;
5. execute one of the six supported fan/light/curtain actions;
6. post ACK only after execution succeeds; and
7. remember the command ID to prevent duplicate execution.

Build and upload:

```bash
pio run
pio run --target upload
pio device monitor
```

**Expected result:** Serial Monitor shows successful telemetry requests, a command is executed once, and ACK changes its state to `Executed`.

## Troubleshooting

- If Wi-Fi connects but HTTP fails, verify `API_BASE_URL`, EC2 public IP changes, port `8000`, and the Security Group.
- If an actuator resets the board, use an appropriate external supply and a common ground.
- Do not ACK unsupported or failed commands; log the reason instead.

Next: [connect the React dashboard](../5.7-Frontend-Integration/).
