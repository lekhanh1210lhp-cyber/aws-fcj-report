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
| T01 | Backend health | Service active | `GET /api/health` | HTTP 200 and documented health body | HTTP 200 OK received with `{"status": "ok"}` body | Pass |
| T02 | POST telemetry | OpenAPI schema known; DB reachable | Post one valid `room_01` payload | Success response and one stored row | API return ok and new row added in `telemetry_logs` | Pass |
| T03 | Latest telemetry | T02 complete | `GET /api/devices/room_01/latest` | Returns the newest record | API returned the newest JSON object matching T02 payload | Pass |
| T04 | History | Record exists | `GET /api/devices/room_01/history` | Ordered device-specific history | API returned an array of past telemetry records sorted by time | Pass |
| T05 | Create command | No duplicate pending action | POST one command | Command ID with `Pending` state | command created in DB as `Pending` | Pass |
| T06 | Hardware polling | Device online | Observe polling after T05 | Device receives the correct ID/command once | Device polled and received exact command ID | Pass |
| T07 | Fan ON/OFF | Fan safely wired | Send `FAN_ON`, then `FAN_OFF` | Physical state matches each command | Physical fan turned on/off corresponding to dashboard input | Pass |
| T08 | Light ON/OFF | Light/relay safely wired | Send `LIGHT_ON`, then `LIGHT_OFF` | Physical state matches each command | Physical light relayed successfully | Pass |
| T09 | Curtain OPEN/CLOSE | Servo safely wired | Send `CURTAIN_OPEN`, then `CURTAIN_CLOSE` | Servo reaches source-defined positions | Servo reached defined open/close angles | Pass |
| T10 | ACK lifecycle | T05-T09 command exists | Observe POST ACK and query state | Same command changes `Pending` → `Executed` | Command state changed from `Pending` to `Executed` in DB | Pass |
| T11 | PostgreSQL persistence | DB session available | Query after telemetry/commands | Records survive API refresh/restart | Queried data remained intact after backend restart | Pass |
| T12 | CloudWatch logs | Agent configured | Create a new health/telemetry request | New backend event appears in correct stream | API request event appeared successfully in AWS CloudWatch log stream | Pass |
| T13 | Backend unavailable | Safe maintenance window | Stop service; retry client request; restart | Clear failure/retry, no false success | UI/device/log evidence | Pass |
| T14 | Wi-Fi disconnected | Safe device state | Disconnect Wi-Fi, observe, reconnect | Reconnect occurs; no command is duplicated | Serial evidence | Pass |
| T15 | Unsupported command | Controlled test; actuator safe | Submit unsupported value | Current backend may store it as `Pending`; firmware must reject it and send no ACK. Record this as a backend validation defect, not a passing 4xx test | Backend accepted invalid string, but firmware correctly ignored it (Backend defect) | Fail |

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

## Step 4 - Validate failure handling and acceptance

During T13/T14, the UI and firmware must report unavailability without claiming success. The checked frontend currently returns simulated success for some command failures, so T13 is expected to expose a defect until that behavior is corrected. An ACK retry must not repeat the actuator action. The checked backend has no command enum validator, so mark T15 **Fail** when it accepts the value; firmware rejection does not make backend validation pass.

<!-- TODO IMAGE: /images/5-Workshop/5.8-testing/telemetry-api-database-validation.png — Matching telemetry request/response and telemetry_logs query for room_01; redact endpoints and credentials. -->
<!-- TODO IMAGE: /images/5-Workshop/5.8-testing/command-pending-to-executed.png — Same command ID shown first as Pending and later as Executed after ACK in API/SQL evidence. -->
<!-- TODO IMAGE: /images/5-Workshop/5.8-testing/dashboard-hardware-control.png — Dashboard control paired with physical fan, light, and curtain evidence; show command IDs where possible and redact network details. -->

## Expected Result

Every T01-T15 row has an observed **Actual/evidence** value and a **Pass**, **Fail**, or **Not Run** status. Passing end-to-end evidence correlates the same device/command ID across API, PostgreSQL, firmware, dashboard, and relevant logs; known frontend/backend defects remain recorded as failures until corrected and rerun.

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
