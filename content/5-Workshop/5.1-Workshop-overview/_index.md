---
title: "Workshop Overview"
date: "2026-07-28"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Context and problem

Small rooms and laboratories often operate sensors and actuators independently. Readings are not retained centrally, users cannot review history, and a dashboard click does not prove that the physical actuator completed the action. This workshop addresses that gap for one sample room without presenting the prototype as a production building-management system.

## Target Users and Proposed Solution

| Target user | Need | Workshop value |
| :--- | :--- | :--- |
| Workshop participant / cloud learner | Deploy and validate an end-to-end AWS workload | A reproducible path from AWS infrastructure to application, hardware, monitoring, and evidence |
| Room operator | View current/history data and request actuator changes | One dashboard for telemetry and fan, light, curtain, and mode commands |
| Maintainer / developer | Trace faults across software, database, network, and hardware | Correlated API, SQL, systemd, Serial Monitor, and CloudWatch evidence |
| Project reviewer / FCAJ mentor | Assess AWS relevance, implementation depth, and individual work | Explicit architecture decisions, measurable tests, trade-offs, and handover links |

YOLO UNO sends environmental telemetry through HTTP to FastAPI on EC2. FastAPI persists telemetry and command state in PostgreSQL. The dashboard reads latest/history data and creates commands; the device polls, executes, and acknowledges them.

## Relevance to FCAJ and AWS

The workshop matches FCAJ learning outcomes by connecting cloud architecture, Linux operations, networking, security, database integration, full-stack development, physical IoT, testing, monitoring, documentation, and handover in one traceable project. AWS is not used only as generic hosting: EC2, EBS, RDS, VPC, Security Groups, an IAM Role, CloudWatch, and CloudWatch Alarms each have a documented responsibility, operational check, cost consideration, and architectural trade-off.

The scope also demonstrates engineering judgment. It reuses the implemented FastAPI/PostgreSQL/HTTP design instead of claiming unimplemented serverless or managed-IoT services, and it records production gaps as future work.

## Technical objectives

1. Ingest telemetry from physical YOLO UNO hardware.
2. Retrieve the latest record and time-ordered history for `room_01`.
3. Control operating mode, fan, light, and curtain using eight firmware-supported commands.
4. Make command completion observable through `Pending` to `Executed` and ACK.
5. Run the backend under `systemd` and monitor EC2, RDS, and logs.
6. Leave a reproducible bilingual runbook and evidence checklist.

## Scope

| In scope | Out of scope in the current implementation |
| :--- | :--- |
| One sample device: `room_01` | Enterprise BMS and multi-tenant operations |
| DHT20 temperature/humidity | High Availability, Auto Scaling, or a Load Balancer |
| Raw analog light sensor value | Calibrated Lux unless firmware proves the conversion |
| Fan, light/relay, curtain servo | HTTPS and authentication |
| FastAPI, RDS PostgreSQL, React/Vite | AWS IoT Core, Lambda, API Gateway, S3, SNS |
| EC2/EBS, VPC/SG, IAM, CloudWatch | ECS/ECR, Cognito, CloudFront, DynamoDB |

## Functional contract

| Capability | Observable result |
| :--- | :--- |
| Telemetry ingestion | A valid request creates a PostgreSQL telemetry record |
| Latest telemetry | The latest record for `room_01` is returned |
| History | Ordered records for `room_01` are returned |
| Fan control | `FAN_ON` and `FAN_OFF` are accepted and executed |
| Light control | `LIGHT_ON` and `LIGHT_OFF` are accepted and executed |
| Curtain control | `CURTAIN_OPEN` and `CURTAIN_CLOSE` are accepted and executed |
| Operating mode | `MODE_AUTO` enables firmware threshold control; `MODE_MANUAL` disables it |
| Command lifecycle | New command is `Pending`; successful ACK makes it `Executed` |
| CloudWatch monitoring | Configured logs/metrics arrive and alarms evaluate thresholds |

The source contains two rule-based mechanisms, not an AI model: the frontend creates time/threshold recommendations, while firmware Auto mode directly controls the fan at `temperature >= 30°C`, the light at analog value `< 350`, and the curtain around the `< 700` threshold. Direct actuator commands switch firmware to Manual mode.

## Concrete Outputs

| Output | Concrete artifact or evidence |
| :--- | :--- |
| AWS foundation | Resource inventory for EC2/EBS, RDS, VPC/subnets, Security Groups, IAM Role, and CloudWatch |
| Running backend | Active `aws-iot-backend` service, HTTP 200 health response, and deployment commit |
| PostgreSQL persistence | `devices`, `telemetry_logs`, and `commands` table/query evidence |
| Integrated YOLO UNO | Wiring/GPIO record, PlatformIO build, telemetry, command execution, and ACK serial evidence |
| Local dashboard | Latest/history view, control request, returned command ID/state, and clear real/simulated source |
| Monitoring | Backend/error log streams, EC2/RDS metrics, and six documented alarm configurations |
| Validation package | Completed T01-T15 matrix with Pass/Fail/Not Run, evidence links, owners, and reruns |
| Handover package | Bilingual Workshop, known limitations, source/deployment commit IDs, and signed checklist |

## Measurable Success Criteria

| ID | Criterion | Measurement |
| :--- | :--- | :--- |
| S01 | Backend availability | `GET /api/health` returns HTTP 200 from the deployed service |
| S02 | Telemetry persistence | One valid `room_01` POST produces one identifiable row in `telemetry_logs` |
| S03 | Dashboard retrieval | Latest and ordered history requests return the stored `room_01` data |
| S04 | Physical control | All six direct actuator commands are tested once with command ID and physical evidence |
| S05 | Mode control | `MODE_AUTO` and `MODE_MANUAL` are observed in firmware behavior/serial evidence |
| S06 | Command completion | The same command ID is captured first as `Pending` and later as `Executed` after ACK |
| S07 | Monitoring | Both backend log groups, required EC2/RDS metrics, and six alarm definitions are visible |
| S08 | Reproducibility and safety | Another participant follows the runbook without exposed credentials; all T01-T15 rows have a recorded status |

The criteria define acceptance; they do not claim that a test passed until the referenced evidence is attached.

<!-- TODO IMAGE: /images/5-Workshop/5.1-overview/end-to-end-system-overview.png — End-to-end view showing the dashboard, EC2 backend, RDS command state, and YOLO UNO hardware without credentials. -->

## Troubleshooting checkpoint

If the team cannot state which component owns a failure, trace one request across browser Network, FastAPI logs, PostgreSQL, device Serial Monitor, and ACK status. Do not label an unverified result as passed.

Next: [prepare the required account, tools, and hardware](../5.2-Prerequisites/).
