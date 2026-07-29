---
title: "Results, Challenges and Future Improvements"
date: "2026-07-28"
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

## Overview

This section separates source-verified implementation from execution evidence still required. The application source was reviewed in `F:\aws-iot-dashboard`; deployment exports, CloudWatch screenshots, database captures, and physical test artifacts are not present in this report repository. The items below remain acceptance statements to confirm against section 5.8, not fabricated test results.

## Results to confirm

| Result | Acceptance evidence |
| :--- | :--- |
| Telemetry end to end | YOLO UNO request, FastAPI response/log, RDS row, dashboard latest view |
| Dashboard and history | Latest card, ordered history/chart, loading/error behavior |
| PostgreSQL persistence | Telemetry and command queries before/after API refresh |
| Fan/light/curtain control | Command IDs paired with physical-device evidence |
| `Pending` → `Executed` | Create response and later query for the same command ID |
| Command ACK | Device serial line, backend log, and stored final state |
| CloudWatch monitoring | Recent backend log, EC2/RDS metrics, and alarm configuration |

Mark a result complete only when its evidence is attached. This prototype does not prove HA, sub-50 ms latency, fail-proof behavior, HTTPS, authentication, or AI control.

## Source-review findings

- Backend command input has no supported-command validator; unsupported values can be stored as `Pending`.
- Command polling returns the oldest pending record (FIFO), although the route is named `commands/latest`.
- ACK lookup uses command ID without verifying the route device ID or prior state.
- `DEVICE_API_KEY` has a default setting but the routes do not enforce it.
- Frontend fetch failures can switch to `SIMULATED` data, while command failures can still be presented as successful mock state.
- Frontend mode is local state and has no API-backed confirmation; “AI” and “FAIL-PROOF” labels overstate rule-based/demo behavior.
- Frontend labels the raw ADC light reading as Lux and hard-codes a real EC2 target in Vite configuration.
- Hardware prose in the source repository mentions servo GPIO 8 and omits LCD in one place, but active firmware uses GPIO 38 and includes LCD1602. Active code is the workshop authority.

## Challenges and lessons learned

| Problem | Root cause | Solution | Lesson learned |
| :--- | :--- | :--- | :--- |
| SSH key rejected on Windows | Key path/ACL or wrong login user | Use correct AMI user and restrict local key access | Diagnose identity before changing network rules |
| Environment command mismatch | PowerShell, CMD, and Bash use different syntax | Use `$env:...`, `%...%`, and `$HOME` only in their own shell | Label every command environment |
| Port 8000 unreachable | SG closed or Uvicorn bound to `127.0.0.1` | Open approved source and bind `0.0.0.0` for demo | Test local health before public path |
| RDS SSL failure | Wrong CA path, hostname, or `DATABASE_URL` | Use current bundle and absolute path; verify endpoint | Network success and TLS success are separate |
| `systemd` service fails | Wrong user/path/module/environment | Inspect status/journal and mirror successful manual run | Promote a verified command into the unit |
| Vite proxy 404/CORS | Wrong target/path or proxy bypass | Use relative `/api`, restart Vite, inspect Network | Keep one API base configuration |
| Public IP changes | EC2 stop/start assigns a different address | Update local/device configuration | Stable endpoint remains future work |
| Endpoint mismatch | Singular/plural routes or stale client contract | Treat OpenAPI and source as canonical | Share one versioned API contract |
| Duplicate command | Poll/refresh/retry submits or executes twice | Check pending state and last command ID | ACK retry must not repeat actuation |
| ACK timing hides `Pending` | Fast polling executes immediately | Preserve create response and final state with same ID | Evidence must follow entity IDs |
| Light reading is inaccurate | Raw ADC lacks calibration | Label as analog value and calibrate later | Do not invent Lux |
| CloudWatch Agent has no data | IAM, path, dimension, or config error | Check agent log and actual log source | Permission and collection configuration are distinct |

## Future improvements

- Attach an Elastic IP or use a domain for a stable endpoint.
- Add Nginx or another reviewed reverse proxy.
- Add HTTPS, authentication, and stronger authorization.
- Store application secrets in a managed secret solution.
- Support more devices and rooms with ownership/authorization rules.
- Evaluate WebSocket or MQTT for lower-overhead updates.
- Evaluate AWS IoT Core as a future messaging option; it is not deployed now.
- Containerize where useful and define infrastructure with code.
- Automate tested deployment and rollback.
- Add reviewed alarm notifications.
- Calibrate the light sensor and publish the conversion method/unit.

Each future item needs an owner, architecture review, cost/security analysis, implementation, and tests before it can move into the current-state diagram.

## Result

After evidence review, record Passed, Failed, or Not Run for each acceptance statement and link the owning issue for any gap. Never convert “expected” into “achieved” without proof.

Next: [prepare the project handover](../5.12-Project-Handover/).
