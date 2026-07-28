---
title: "Architecture and Service Design"
date: "2026-07-28"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Architecture

![AWS IoT Monitoring and Control Dashboard architecture](/images/5-Workshop/5.3-architecture/aws-iot-dashboard-architecture.png)

*Figure 5-2. The architecture image copied from the application repository shows the service boundaries, EC2-to-RDS connection, device HTTP paths, and CloudWatch monitoring path.*

The dashboard user, local React + Vite frontend, and YOLO UNO are outside AWS. Inside the AWS Cloud, the VPC contains a public subnet with EC2 and a DB Subnet Group with RDS. EBS is the EC2 root volume. The EC2 and RDS Security Groups control traffic. The IAM Role belongs to the AWS account, while CloudWatch is a regional service; neither is placed inside the VPC.

## Components and AWS service selection

| Component/service | Responsibility and reason |
| :--- | :--- |
| React + Vite + TypeScript + Tailwind CSS | Local operator UI, telemetry views, controls, and rule-based recommendations |
| Amazon EC2 | Full control of FastAPI, Python, Uvicorn, and `systemd` |
| Amazon EBS | Persistent root volume attached to EC2 |
| Amazon RDS for PostgreSQL | Managed relational persistence for telemetry and command state |
| Amazon VPC and subnets | Network boundaries for EC2 and the DB Subnet Group |
| Security Groups | Stateful allow rules for SSH/API and EC2-to-RDS traffic |
| AWS IAM Role | Supplies temporary permission for EC2 to publish monitoring data |
| CloudWatch Agent | Software on EC2 that reads guest metrics/log files |
| Amazon CloudWatch/Alarms | Stores metrics/logs and evaluates thresholds |
| YOLO UNO / ESP32-S3 | Reads sensors, controls actuators, polls commands, and sends ACK |

An IAM Role grants permission; it is not the CloudWatch Agent. The agent is a process installed and managed on EC2.

## Verified API contract

FastAPI source in `backend/main.py` and `backend/app/api/` confirms these routes:

| Method | Route | Consumer |
| :--- | :--- | :--- |
| `GET` | `/` | Basic service information |
| `GET` | `/api/health` | Health check |
| `POST` | `/api/telemetry` | YOLO UNO telemetry |
| `GET` | `/api/devices/{device_id}/latest` | Dashboard latest view |
| `GET` | `/api/devices/{device_id}/history` | Dashboard history view |
| `POST` | `/api/devices/{device_id}/commands` | Dashboard control |
| `GET` | `/api/devices/{device_id}/commands/latest` | Device polling |
| `POST` | `/api/devices/{device_id}/commands/{command_id}/ack` | Device ACK |

Firmware supports `MODE_AUTO`, `MODE_MANUAL`, `FAN_ON`, `FAN_OFF`, `LIGHT_ON`, `LIGHT_OFF`, `CURTAIN_OPEN`, and `CURTAIN_CLOSE`. The backend currently accepts any command string because `DeviceCommand` has no active enum validator; unsupported commands remain `Pending` because firmware rejects them without ACK. Do not use the singular `/api/device/...` form.

## Data flows

1. **Telemetry:** YOLO UNO sends camelCase fields → Pydantic aliases map them to snake_case → SQLAlchemy writes `telemetry_logs` in RDS → latest/history API → dashboard.
2. **Command:** dashboard → create command → backend writes `commands.state = "Pending"` → the route named `commands/latest` actually returns the oldest pending command first (FIFO) → hardware executes it.
3. **ACK:** device sends command ID → backend changes that command to `Executed` → later telemetry reports actuator state. The current ACK service looks up by command ID only; device ownership validation is a known hardening task.
4. **Monitoring:** EC2 default metrics and agent data/logs → CloudWatch; RDS publishes service metrics; alarms evaluate configured thresholds.

Polling can make `Pending` visible for only a short time. Evidence should therefore include the create-command response and the later `Executed` database/API state.

The database models define `devices`, `telemetry_logs`, and `commands`. Telemetry fields are `temperature`, `humidity`, `light_intensity`, `fan_status`, `light_status`, `curtain_status`, and `timestamp`.

## Network and security design

| Source | Destination | Port | Rule |
| :--- | :--- | :---: | :--- |
| `<ADMIN_IP>/32` | EC2 Security Group | 22 | SSH administration only |
| Demo clients | EC2 Security Group | 8000 | HTTP demo API; avoid `0.0.0.0/0` beyond the demo |
| EC2 Security Group | RDS Security Group | 5432 | PostgreSQL only |
| EC2/RDS | CloudWatch | HTTPS | Outbound monitoring path |

RDS should not be publicly accessible. Secrets stay in ignored local files; EC2 uses an IAM Role instead of hard-coded AWS keys. The current design does not claim HTTPS, authentication, HA, Multi-AZ, a load balancer, or rate limiting.

## Scalability and limitations

The `device_id` route structure and relational schema can support more rooms, but the present acceptance scope is `room_01`. A single EC2 endpoint and periodic HTTP polling are simple for a prototype but create a changing-public-IP risk, polling delay, and a single compute failure domain. Managed messaging, authentication, HTTPS, multiple instances, and Infrastructure as Code are future design options, not deployed features.

## Expected result and troubleshooting

Every arrow in the architecture should map to one API call, network rule, database action, or metric/log path. If a connection is unclear, identify its source, destination, port, identity, and expected evidence before provisioning.

Next: [build the AWS foundation](../5.4-AWS-Infrastructure-Setup/).
