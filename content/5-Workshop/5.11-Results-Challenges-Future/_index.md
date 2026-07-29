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

## Project Customizations

The project is not an unchanged tutorial deployment. Its reviewed customizations include:

- a `room_01` domain model joining physical telemetry, dashboard history, and actuator state;
- a FastAPI/PostgreSQL command lifecycle with stored `Pending` and `Executed` states plus device ACK;
- eight firmware commands covering automatic/manual mode and direct fan, light, and curtain control;
- firmware thresholds, GPIO mapping, LCD1602 output, reconnect timing, and ESP32 Preferences-based ACK recovery;
- a React/Vite dashboard with telemetry charts, controls, rule-based recommendations, and explicit real/simulated-source concerns;
- a private RDS network path through Security Group reference rather than public database access;
- project-specific CloudWatch namespace, two backend log groups, and six documented alarms; and
- an evidence-first bilingual Workshop that separates source-verified behavior from results still requiring screenshots/tests.

These choices adapt the architecture to the implemented source and YOLO UNO hardware. Auto Scaling, Amazon SQS, AWS IoT Core, and an event-driven architecture remain future options and are not project customizations already deployed.

## Individual Contributions

| Contributor | Owned scope and concrete contribution | Evidence path |
| :--- | :--- | :--- |
| **Pham Le Minh Khoi** | AWS architecture, network/security boundaries, EC2/RDS/CloudWatch operations, YOLO UNO wiring, sensors/actuators, telemetry polling, command execution, and ACK | [Architecture](../5.3-Architecture-and-Service-Design/), [AWS setup](../5.4-AWS-Infrastructure-Setup/), [hardware](../5.6-Hardware-Integration/), [CloudWatch](../5.9-CloudWatch-Monitoring/) |
| **Ngo Minh Thuan** | FastAPI routes, Pydantic aliases, SQLAlchemy models, PostgreSQL persistence, telemetry service, command lifecycle, and ACK processing | [API/data design](../5.3-Architecture-and-Service-Design/), [backend/database](../5.5-Backend-and-Database/), [test matrix](../5.8-End-to-End-Testing/) |
| **Thuong Dinh Hung** | React/Vite dashboard, telemetry visualization, control requests, mode/recommendation UI, integration debugging, and demo-video production | [frontend integration](../5.7-Frontend-Integration/), [end-to-end validation](../5.8-End-to-End-Testing/), [handover](../5.12-Project-Handover/) |
| **Le Bao Khanh** | Proposal/report content, blogs/worklogs/events, bilingual Workshop structure, source-to-document verification, navigation, screenshot plan, and QA | [Workshop overview](../5.1-Workshop-overview/), [test/evidence plan](../5.8-End-to-End-Testing/), [results](../5.11-Results-Challenges-Future/), [handover](../5.12-Project-Handover/) |

Contribution is accepted only when the linked section is paired with source commit, screenshot, log, test record, document history, or other attributable evidence. This table records ownership; it does not replace the individual reflections below.

## Individual Reflections

### Pham Le Minh Khoi

| Reflection field | Reflection |
| :--- | :--- |
| Challenge | Integrate a publicly reachable demo backend, private PostgreSQL, monitoring, and physical actuators without confusing cloud success with hardware success |
| Root Cause | The flow crosses VPC rules, IAM, Linux services, HTTP polling, electrical wiring, and asynchronous ACK state |
| Solution | Use an EC2-to-RDS Security Group reference, EC2 IAM Role, systemd/CloudWatch checks, source-defined GPIOs, safe power, command IDs, and persistent ACK recovery |
| Lesson Learned | Validate each boundary independently and correlate one command ID through API, database, serial output, actuator action, and monitoring |
| Future Improvement | Add HTTPS/stable endpoint, Infrastructure as Code, stronger IAM scoping, calibrated hardware evidence, and evaluate managed MQTT only after architecture review |

### Ngo Minh Thuan

| Reflection field | Reflection |
| :--- | :--- |
| Challenge | Preserve telemetry and make command completion observable across polling and ACK |
| Root Cause | Asynchronous clients and database state can diverge; the current source also lacks command enum validation and strict ACK device ownership |
| Solution | Model devices, telemetry, and commands in PostgreSQL; return command IDs/states; use FIFO pending polling and explicit ACK transitions |
| Lesson Learned | An OpenAPI contract and stored state improve traceability, but validation, authorization, idempotency, and schema migration must be designed explicitly |
| Future Improvement | Add supported-command validation, authenticated device identity, device-bound ACK checks, idempotency rules, Alembic migrations, and automated API tests |

### Thuong Dinh Hung

| Reflection field | Reflection |
| :--- | :--- |
| Challenge | Present live telemetry and controls while accurately distinguishing request acceptance, physical execution, and simulated fallback |
| Root Cause | The current frontend polls multiple endpoints, keeps mode locally, falls back to generated data, and can report mock success after a failed command |
| Solution | Inspect DevTools Network, use plural relative API routes, expose command ID/state, label simulated data, and verify physical execution through ACK/evidence |
| Lesson Learned | A responsive UI is not enough; operational truth must come from backend/device state and error handling must never imply unverified success |
| Future Improvement | Remove false-success fallback, add API-backed mode/command status, centralize environment configuration, correct the Lux label, and add component/integration tests |

### Le Bao Khanh

| Reflection field | Reflection |
| :--- | :--- |
| Challenge | Turn evolving and sometimes inconsistent source notes into a coherent bilingual Workshop without inventing deployment evidence |
| Root Cause | Old Workshop pages described unrelated services, prose disagreed with active firmware, and required screenshots/test artifacts were incomplete |
| Solution | Treat active source as authority, align English/Vietnamese structure, document limitations, use exact TODO evidence paths, and run Hugo/structure/link checks |
| Lesson Learned | Technical documentation must distinguish implemented, proposed, expected, and proven states while keeping commands, names, paths, and translations synchronized |
| Future Improvement | Add CI for Hugo/link/secret/parity checks, replace TODOs with attributable evidence, maintain a versioned API/GPIO contract, and schedule member review/sign-off |

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
