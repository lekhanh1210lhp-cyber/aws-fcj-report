---
title: "Workshop Overview"
date: "2026-07-28"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Context and problem

Small rooms and laboratories often operate sensors and actuators independently. Readings are not retained centrally, users cannot review history, and a dashboard click does not prove that the physical actuator completed the action. This workshop addresses that gap for one sample room without presenting the prototype as a production building-management system.

## Users and proposed solution

The primary users are a workshop participant deploying the stack, an operator viewing the dashboard, and a maintainer investigating failures. YOLO UNO sends environmental telemetry through HTTP to FastAPI on EC2. FastAPI persists telemetry and command state in PostgreSQL. The dashboard reads latest/history data and creates commands; the device polls, executes, and acknowledges them.

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

## Success criteria and deliverables

The workshop succeeds when telemetry reaches RDS, the dashboard reads it, each supported actuator command is executed once, ACK updates the stored state, CloudWatch receives the configured evidence, and another participant can reproduce the procedure without real credentials in the documentation.

Deliverables are the AWS resources, deployed backend service, database records, configured firmware, local dashboard, test evidence, CloudWatch views, bilingual workshop, and handover checklist.

<!-- TODO IMAGE: /images/5-Workshop/5.1-overview/end-to-end-system-overview.png — End-to-end view showing the dashboard, EC2 backend, RDS command state, and YOLO UNO hardware without credentials. -->

## Troubleshooting checkpoint

If the team cannot state which component owns a failure, trace one request across browser Network, FastAPI logs, PostgreSQL, device Serial Monitor, and ACK status. Do not label an unverified result as passed.

Next: [prepare the required account, tools, and hardware](../5.2-Prerequisites/).
