---
title: "End-to-End Testing and Validation"
date: "2026-07-28"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Overview and objectives

Validate each system boundary independently before executing complete end-to-end telemetry and command workflows. The verified backend and firmware sources define the expected API schemas and command behavior. Before testing, confirm the deployed FastAPI version through `/docs` or `/openapi.json`.

---

## Step 1 - Establish the test protocol

1. Record the test date, tester, application commit IDs, firmware build, AWS region, and device ID (`room_01`).
2. Redact credentials, API keys, and private endpoints from all evidence.
3. Capture API requests/responses, backend logs, SQL state, device output, dashboard status, and CloudWatch logs where applicable.
4. Record the observed result in **Actual/Evidence** and assign **Pass**, **Fail**, or **Not Run** only after execution.
5. Restore hardware and services to a safe operating state after failure testing.

---

## Step 2 - Execute and record the test matrix

| ID | Objective | Preconditions | Steps | Expected result | Actual/evidence | Pass/Fail |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| T01 | Backend health | Service active | `GET /api/health` | HTTP 200 and documented health body | HTTP 200 response and backend health log | **Pass** |
| T02 | POST telemetry | OpenAPI schema known; DB reachable | Submit one valid `room_01` telemetry payload | Success response and one stored row | Figure 15: matching API and SQL record | **Pass** |
| T03 | Latest telemetry | T02 complete | `GET /api/devices/room_01/latest` | Returns newest telemetry | Latest API response verified | **Pass** |
| T04 | Telemetry history | Multiple records exist | `GET /api/devices/room_01/history` | Ordered device-specific history | History response and dashboard chart | **Pass** |
| T05 | Create command | No duplicate pending action | POST one supported command | Command ID with `Pending` state | Figure 16: command ID 189 in `Pending` state | **Pass** |
| T06 | Hardware polling | Device online | Observe polling after T05 | Device receives the correct command exactly once | Hardware demonstration video | **Pass** |
| T07 | Fan ON/OFF | Fan safely wired | Send `FAN_ON`, then `FAN_OFF` | Physical fan state matches commands | Dashboard + hardware evidence | **Pass** |
| T08 | Light ON/OFF | Light safely wired | Send `LIGHT_ON`, then `LIGHT_OFF` | Physical light state matches commands | Dashboard + hardware evidence | **Pass** |
| T09 | Curtain OPEN/CLOSE | Servo safely wired | Send `CURTAIN_OPEN`, then `CURTAIN_CLOSE` | Servo rotates to 90° then returns to 0° | Dashboard + hardware evidence | **Pass** |
| T10 | ACK lifecycle | T05-T09 command exists | Observe ACK and query state | Same command changes `Pending` → `Executed` | Figure 16: command ID 189 becomes `Executed` | **Pass** |
| T11 | PostgreSQL persistence | DB available | Query database after telemetry and commands | Records persist across API refresh/restart | SQL evidence | **Pass** |
| T12 | CloudWatch logs | Agent configured | Create new API request | New backend event appears in CloudWatch | CloudWatch logs (Section 5.9) | **Pass** |
| T13 | Backend unavailable | Safe maintenance window | Stop backend, retry request, restart | Client reports failure; no false success | UI, API, and log evidence | **Record** |
| T14 | Wi-Fi disconnected | Safe device state | Disconnect Wi-Fi, reconnect | Device reconnects without duplicating commands | Serial Monitor evidence | **Pass** |
| T15 | Unsupported command | Controlled environment | Submit unsupported command | Backend currently accepts request, firmware rejects execution and sends no ACK. Record as backend validation defect. | API + SQL + Serial Monitor | **Fail (expected)** |

---

## Step 3 - API and database verification

From EC2 Linux Bash:

```bash
curl -i http://127.0.0.1:8000/api/health
curl -s http://127.0.0.1:8000/api/devices/room_01/latest
curl -s http://127.0.0.1:8000/api/devices/room_01/history
```

Create telemetry using the camelCase fields defined in Section 5.6. Create a command using:

```json
{
  "command": "FAN_ON"
}
```

The Pydantic model also supports the alias `Command`; however, `populate_by_name=True` allows the lowercase field name. A device record should already exist, typically created by the first telemetry request.

Verify command states in PostgreSQL:

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

Because polling devices may acknowledge commands almost immediately, preserve the POST response showing the initial `Pending` state before capturing the final `Executed` record with the same command ID.

---

### T02 evidence - Telemetry persisted in Amazon RDS

Figure 15 demonstrates a controlled `curl` request that isolates FastAPI and PostgreSQL persistence. Hardware integration was validated separately; this figure specifically verifies test case T02.

![Telemetry submitted through the API and stored in PostgreSQL](/images/5-Workshop/5.8-testing/telemetry-api-database-validation.png)

*Figure 15. Telemetry submitted through the REST API and successfully persisted in Amazon RDS for PostgreSQL.*

---

### T05/T10 evidence - Command lifecycle

To isolate backend behavior while hardware was unavailable during evidence collection, the `FAN_ON` command was created through the REST API and acknowledged manually through the ACK endpoint. The same command ID (`189`) transitions from `Pending` to `Executed`, validating the backend and database workflow independently of hardware.

![Command 189 changing from Pending to Executed after ACK](/images/5-Workshop/5.8-testing/command-pending-to-executed.png)

*Figure 16. Validation of the command lifecycle from Pending to Executed through the FastAPI ACK endpoint.*

---

### T06-T09 evidence - Dashboard and hardware validation

The following dashboard screenshots demonstrate successful remote command submission for the fan, lighting, and curtain subsystems. Corresponding physical actuator responses are recorded in the demonstration video.

![Remote fan control through dashboard](/images/5-Workshop/5.8-testing/dashboard-hardware-control_Fan.PNG)

![Remote light control through dashboard](/images/5-Workshop/5.8-testing/dashboard-hardware-control_LED.PNG)

![Remote curtain control through dashboard](/images/5-Workshop/5.8-testing/dashboard-hardware-control_Curtain.PNG)

*Figure 17. Dashboard interface used to issue remote control commands for the fan, lighting, and curtain subsystems.*

The complete hardware demonstration is available in the Google Drive video below:

https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing

The recording provides end-to-end evidence that the dashboard commands are received by the ESP32 firmware and correctly actuate the physical fan, light, and curtain mechanisms.

---

## Step 4 - Failure handling and acceptance

Failure scenarios (T13-T15) verify that the system reports errors correctly without producing false-positive success states.

- During backend downtime (T13), the UI and firmware should report service unavailability instead of indicating successful command execution.
- During Wi-Fi interruption (T14), the firmware must reconnect automatically and resume polling without executing duplicate commands.
- Unsupported commands (T15) should ideally be rejected by backend validation. Since the current backend does not enforce a command enumeration, acceptance of unsupported values should be recorded as a backend validation defect rather than a successful test.

---

## Expected result

Every test case (T01-T15) should contain:

- Actual observed evidence
- Pass / Fail / Not Run status
- Supporting API responses, SQL queries, hardware output, dashboard screenshots, or CloudWatch logs

Successful end-to-end validation correlates the same device ID and command ID consistently across the REST API, PostgreSQL, firmware, dashboard, and monitoring logs.

---

## Troubleshooting

This section defines execution guidance rather than claiming test completion. Do not describe these tests as stress or performance testing, and do not report latency or throughput values unless they have been measured separately.

| Symptom | Check |
| :--- | :--- |
| `Pending` state not visible | Preserve the POST response before ACK and query the same command ID afterwards |
| Dashboard and database disagree | Verify API endpoint, dashboard data source, and latest database record |
| Duplicate command execution | Compare command IDs, `lastAck`, and `pendingAck`; distinguish ACK retry from actuator execution |
| Backend reports success during failure | Verify frontend error handling and backend availability |
| Unsupported command accepted | Record as backend validation defect until command validation is implemented |
| Test cannot be reproduced | Record commit IDs, AWS region, device ID, timestamp, and exact preconditions |
| Evidence contains sensitive information | Redact and recapture before publication |

Next: [Configure and Validate CloudWatch](../5.9-CloudWatch-Monitoring/).