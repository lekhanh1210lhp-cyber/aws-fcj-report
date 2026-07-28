---
title: "End-to-End Testing and Validation"
date: "2026-07-28"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

# End-to-End Testing and Validation

Run tests in order so each failure has a narrow cause.

| ID | Test | Expected result | Evidence |
|---|---|---|---|
| T01 | `GET /api/health` | HTTP 200 | curl or Swagger |
| T02 | `POST /api/telemetry` | New RDS row | response and SQL |
| T03 | Latest/history request | `room_01` data returned | JSON and chart |
| T04 | Create command | Status is `Pending` | response and SQL |
| T05 | Device polling | Correct actuator moves once | Serial Monitor/video |
| T06 | Device ACK | Status becomes `Executed` | SQL before/after |
| T07 | CloudWatch log/metric | New event/datapoint appears | Console screenshot |

## Validation sequence

1. Save the health response.
2. Post a known telemetry sample and query its row in PostgreSQL.
3. Confirm latest and history contain the same values.
4. Create a supported command and capture its `Pending` row.
5. Start hardware polling, observe physical execution, and capture the ACK.
6. Query the same command and confirm `Executed`.

Also test backend stopped, incorrect endpoint, Wi-Fi loss, RDS connection failure, and an unsupported command. The UI and firmware must fail visibly without claiming execution.

**Expected result:** API, database, dashboard, hardware, and monitoring evidence tell one consistent story.

## Troubleshooting

- Use one command ID across request, database, Serial Monitor, and ACK evidence.
- If a test is intermittent, record timestamps and compare backend, device, and CloudWatch logs before repeating it.

Next: [configure CloudWatch monitoring](../5.9-CloudWatch-Monitoring/).
