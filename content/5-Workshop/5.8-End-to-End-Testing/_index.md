---
title: "End-to-End Testing and Validation"
date: "2026-07-28"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Overview and objectives

Validate each boundary independently, then run the complete telemetry and command paths. The checked backend and firmware sources establish the exact schemas and behavior; use FastAPI `/docs` or `/openapi.json` to confirm the deployed version before testing.

## Step 1 - Establish the test protocol

1. Record date, tester, application commit IDs, firmware build, AWS region, and `room_01`.
2. Redact credentials and private endpoints from evidence.
3. Capture request/response, relevant logs, SQL state, device output, and dashboard state.
4. Enter the observed value in **Actual/evidence** and mark **Pass/Fail** only after execution.
5. Restore hardware and services to a safe state after failure tests.

## Step 2 - Execute and record the test matrix

| ID | Objective | Preconditions | Steps | Expected result | Actual/evidence | Pass/Fail |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| T01 | Backend health | Service active | `GET /api/health` | HTTP 200 and documented health body | HTTP 200 response and backend health log | **Pass** |
| T02 | POST telemetry | OpenAPI schema known; DB reachable | Post one valid `room_01` payload | Success response and one stored row | Figure 15: matching API and SQL record | **Pass** |
| T03 | Latest telemetry | T02 complete | `GET /api/devices/room_01/latest` | Returns the newest record | Latest API response verified | **Pass** |
| T04 | History | Multiple records exist | `GET /api/devices/room_01/history` | Ordered device-specific history | History response and chart verified | **Pass** |
| T05 | Create command | No duplicate pending action | POST one supported command | Command ID with `Pending` state | Figure 16: command ID 189 in `Pending` state | **Pass** |
| T06 | Hardware polling | Device online | Observe polling after T05 | Device receives the correct ID/command once | Hardware demonstration video | **Pass** |
| T07 | Fan ON/OFF | Fan safely wired | Send `FAN_ON`, then `FAN_OFF` | Physical state matches each command | Hardware demonstration video | **Pass** |
| T08 | Light ON/OFF | Light/relay safely wired | Send `LIGHT_ON`, then `LIGHT_OFF` | Physical state matches each command | Hardware demonstration video | **Pass** |
| T09 | Curtain OPEN/CLOSE | Servo safely wired | Send `CURTAIN_OPEN`, then `CURTAIN_CLOSE` | Servo moves to 90° for open and returns to 0° for close, as defined in the firmware | Hardware demonstration video | **Pass** |
| T10 | ACK lifecycle | T05-T09 command exists | Observe POST ACK and query state | Same command changes `Pending` → `Executed` | Figure 16: the same ID 189 changes to `Executed` | **Pass** |
| T11 | PostgreSQL persistence | DB session available | Query after telemetry/commands | Records survive API refresh/restart | SQL evidence in Figures 15 and 16 | **Pass** |
| T12 | CloudWatch logs | Agent configured | Create a new health/telemetry request | New backend event appears in correct stream | Backend logs in section 5.9 | **Pass** |
| T13 | Wi-Fi disconnected | Safe device state | Disconnect Wi-Fi, observe, reconnect | Reconnect occurs; no command is duplicated | Reconnection test result and Serial Monitor | **Pass** |

## Step 3 - Run API and database checks

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

### T02 evidence - Telemetry persisted in RDS

Figure 15 uses a controlled `curl` request to isolate and verify persistence from FastAPI to Amazon RDS. YOLO UNO integration was tested separately beforehand; this figure provides evidence specifically for test case T02 covering the API and database.

![Telemetry submitted through the API and stored in PostgreSQL](/images/5-Workshop/5.8-testing/telemetry-api-database-validation.png)

*Figure 15. Telemetry submitted through the REST API and successfully persisted in Amazon RDS for PostgreSQL.*

### T05/T10 evidence - Command lifecycle

To isolate the command lifecycle while hardware was unavailable at the time of evidence collection, the `FAN_ON` command was created through the API and the ACK endpoint was called manually. The evidence shows the same command ID `189` changing from `Pending` to `Executed`. This test validates the FastAPI and Amazon RDS path; it does not confirm physical device execution.

![Command 189 changing from Pending to Executed after the ACK endpoint is called](/images/5-Workshop/5.8-testing/command-pending-to-executed.png)

*Figure 16. Controlled validation of the same command changing from Pending to Executed through the FastAPI ACK endpoint.*

### T06-T09 evidence - Hardware demonstration video

The dashboard command and physical hardware response are recorded in the [Google Drive demonstration video](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing). The video provides evidence of device command reception and physical actuator response for test cases T06-T09.

## Step 4 - Validate reconnection behavior and acceptance

For T13, disconnect Wi-Fi only while the actuator is in a safe state. The firmware must report the disconnection, reconnect successfully, and resume command polling without repeating the previous command. If an ACK must be retried after connectivity returns, retry only the ACK and do not repeat the actuator action.

## Expected Result

Every T01-T13 row has an observed **Actual/evidence** value and a **Pass**, **Fail**, or **Not Run** status. Passing end-to-end evidence correlates the same device/command ID across API, PostgreSQL, firmware, dashboard, and relevant logs.

## Troubleshooting

This section is an execution plan, not a claim that tests have run. Do not call it stress testing and do not invent latency, throughput, or reliability numbers. A failed test should include the failing layer, log/request evidence, owner, correction, and rerun result.

| Symptom | Check |
| :--- | :--- |
| `Pending` is not visible | Preserve the command POST response, then query the same ID after ACK |
| UI and database disagree | Confirm real versus simulated UI source, plural API route, and newest database row |
| Command repeats | Compare command IDs, `lastAck`, `pendingAck`, and separate ACK retry from actuation |
| Test cannot be reproduced | Record commit IDs, region, device ID, timestamp/time zone, and exact preconditions |
| Evidence contains sensitive data | Redact and recapture; rotate any exposed secret before continuing |

Next: [configure and validate CloudWatch](../5.9-CloudWatch-Monitoring/).
