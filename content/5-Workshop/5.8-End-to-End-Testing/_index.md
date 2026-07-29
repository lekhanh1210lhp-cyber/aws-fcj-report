---
title: "End-to-End Testing and Validation"
date: "2026-07-28"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Overview and objectives

Validate each boundary independently, then run the complete telemetry and command paths. The checked backend and firmware sources establish the exact schemas and behavior; use FastAPI `/docs` or `/openapi.json` to confirm the deployed version before testing.

## Test protocol

1. Record date, tester, application commit IDs, firmware build, AWS region, and `room_01`.
2. Redact credentials and private endpoints from evidence.
3. Capture request/response, relevant logs, SQL state, device output, and dashboard state.
4. Enter the observed value in **Actual/evidence** and mark **Pass/Fail** only after execution.
5. Restore hardware and services to a safe state after failure tests.

## Test matrix

| ID | Objective | Preconditions | Steps | Expected result | Actual/evidence | Pass/Fail |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| T01 | Backend health | Service active | `GET /api/health` | HTTP 200 and documented health body | Record curl response | Record |
| T02 | POST telemetry | OpenAPI schema known; DB reachable | Post one valid `room_01` payload | Success response and one stored row | Attach API + SQL | Record |
| T03 | Latest telemetry | T02 complete | `GET /api/devices/room_01/latest` | Returns the newest record | Attach response | Record |
| T04 | History | Multiple records exist | `GET /api/devices/room_01/history` | Ordered device-specific history | Attach response/chart | Record |
| T05 | Create command | No duplicate pending action | POST one supported command | Command ID with `Pending` state | Attach response | Record |
| T06 | Hardware polling | Device online | Observe polling after T05 | Device receives the correct ID/command once | Serial evidence | Record |
| T07 | Fan ON/OFF | Fan safely wired | Send `FAN_ON`, then `FAN_OFF` | Physical state matches each command | Video/photo + IDs | Record |
| T08 | Light ON/OFF | Light/relay safely wired | Send `LIGHT_ON`, then `LIGHT_OFF` | Physical state matches each command | Video/photo + IDs | Record |
| T09 | Curtain OPEN/CLOSE | Servo safely wired | Send open, then close | Servo reaches source-defined positions | Video/photo + IDs | Record |
| T10 | ACK lifecycle | T05-T09 command exists | Observe POST ACK and query state | Same command changes `Pending` → `Executed` | API + SQL + log | Record |
| T11 | PostgreSQL persistence | DB session available | Query after telemetry/commands | Records survive API refresh/restart | SQL evidence | Record |
| T12 | CloudWatch logs | Agent configured | Create a new health/telemetry request | New backend event appears in correct stream | CloudWatch evidence | Record |
| T13 | Backend unavailable | Safe maintenance window | Stop service; retry client request; restart | Clear failure/retry, no false success | UI/device/log evidence | Record |
| T14 | Wi-Fi disconnected | Safe device state | Disconnect Wi-Fi, observe, reconnect | Reconnect occurs; no command is duplicated | Serial evidence | Record |
| T15 | Unsupported command | Controlled test; actuator safe | Submit unsupported value | Current backend may store it as `Pending`; firmware must reject it and send no ACK. Record this as a backend validation defect, not a passing 4xx test | API + SQL + serial log | Record |

## API and database checks

From EC2 Linux Bash:

```bash
curl -i http://127.0.0.1:8000/api/health
curl -s http://127.0.0.1:8000/api/devices/room_01/latest
curl -s http://127.0.0.1:8000/api/devices/room_01/history
```

Create telemetry with the camelCase fields from 5.6. Create a command with `{ "command": "FAN_ON" }`; the Pydantic field also has the alias `Command`, but `populate_by_name=True` means lowercase `command` is accepted. A device row must already exist, normally created by the first telemetry request. In PostgreSQL `psql`, inspect command state:

```sql
SELECT
    id,
    device_id,
    command,
    state
FROM commands
ORDER BY id DESC
LIMIT 6;
```

A polling device may acknowledge so quickly that `Pending` is missed in a later query. Preserve the POST response showing `Pending`, then capture the final `Executed` record with the same ID.

## Failure handling and acceptance

During T13/T14, the UI and firmware must report unavailability without claiming success. The checked frontend currently returns simulated success for some command failures, so T13 is expected to expose a defect until that behavior is corrected. An ACK retry must not repeat the actuator action. The checked backend has no command enum validator, so mark T15 **Fail** when it accepts the value; firmware rejection does not make backend validation pass.

<!-- TODO IMAGE: /images/5-Workshop/5.8-testing/telemetry-api-database-validation.png — Matching telemetry request/response and telemetry_logs query for room_01; redact endpoints and credentials. -->
<!-- TODO IMAGE: /images/5-Workshop/5.8-testing/command-pending-to-executed.png — Same command ID shown first as Pending and later as Executed after ACK in API/SQL evidence. -->
<!-- TODO IMAGE: /images/5-Workshop/5.8-testing/dashboard-hardware-control.png — Dashboard control paired with physical fan, light, and curtain evidence; show command IDs where possible and redact network details. -->
![Real-time verification of remote fan hardware control through dashboard control panel](\images\5-Workshop\5.8-testing\dashboard-hardware-control_Fan.PNG)

![Real-time verification of remote light control through dashboard control panel](\images\5-Workshop\5.8-testing\dashboard-hardware-control_Light.PNG)

![Real-time verification of remote curtain hardware control through dashboard control panel](\images\5-Workshop\5.8-testing\dashboard-hardware-control_Curtain.PNG)

## Result and troubleshooting

This section is an execution plan, not a claim that tests have run. Do not call it stress testing and do not invent latency, throughput, or reliability numbers. A failed test should include the failing layer, log/request evidence, owner, correction, and rerun result.

Next: [configure and validate CloudWatch](../5.9-CloudWatch-Monitoring/).
