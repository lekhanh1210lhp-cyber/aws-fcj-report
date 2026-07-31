---
title: "Architecture and Service Design"
date: "2026-07-28"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Architecture

![AWS IoT Monitoring and Control Dashboard architecture](/images/5-Workshop/5.3-architecture/aws_diagram.png)

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

## 5.3.3 AWS Service Selection and Architectural Trade-offs

The project selects AWS services based on four primary criteria:

1. Compatibility with the current architecture and source code.
2. Simplicity for implementation and workshop delivery.
3. Direct observability, testing, and operational control.
4. Reasonable cost for a learning and demonstration environment.

Not every serverless service is required for this use case. The current system runs a persistent FastAPI application, connects to PostgreSQL, and communicates with YOLO UNO hardware through HTTP REST APIs. Therefore, the team selected Amazon EC2 and Amazon RDS instead of redesigning the entire platform around Lambda, API Gateway, and DynamoDB.

### Selected AWS Services

| AWS Service | Role in the System | Selection Reason | Trade-off |
| :--- | :--- | :--- | :--- |
| **Amazon EC2** | Hosts the FastAPI backend, Uvicorn, and CloudWatch Agent | Provides full control over the Python environment, dependencies, networking, systemd service, and log files | The team must manage the operating system, packages, services, and part of the security configuration |
| **Amazon EBS** | Provides the EC2 root volume | Persists the operating system, source code, virtual environment, and local logs | An unattached volume may continue generating charges if it is not deleted |
| **Amazon RDS for PostgreSQL** | Stores telemetry and command states | PostgreSQL is suitable for structured telemetry and command records, while RDS reduces database administration effort | A continuously running database instance may cost more than some serverless options at very low usage |
| **Amazon VPC** | Provides the network environment for EC2 and RDS | Enables subnet, routing, and network-access control | Requires correct network and Security Group configuration |
| **Security Groups** | Control inbound traffic to EC2 and RDS | Restrict SSH to the administrator IP and PostgreSQL access to the EC2 Security Group | Incorrect rules may block the application or expose services unnecessarily |
| **AWS IAM Role** | Grants EC2 permission to publish monitoring data | Avoids storing AWS access keys on the instance or in source code | Policies must follow the principle of least privilege |
| **Amazon CloudWatch** | Collects backend logs and infrastructure metrics | Centralizes operational data for troubleshooting and validation | Log ingestion, retention, and custom metrics may generate additional costs |
| **CloudWatch Alarms** | Evaluates CPU, memory, disk, and database-connection thresholds | Provides visibility when configured operating thresholds are exceeded | Alarm usefulness depends on appropriate thresholds and evaluation periods |

### Why Amazon EC2 Was Selected for the FastAPI Backend

The FastAPI backend is a persistent application that exposes REST APIs to both the frontend and the YOLO UNO device. Amazon EC2 allows the team to:

- Install the required Python version and dependencies.
- Run Uvicorn as a `systemd` service.
- Configure backend port `8000`.
- Install CloudWatch Agent.
- Inspect logs and service status through SSH.
- Connect directly to Amazon RDS for PostgreSQL.
- Debug telemetry, command, and acknowledgement requests.

EC2 also makes the workshop easier to demonstrate because participants can observe the backend installation, service management, logging, and troubleshooting steps directly.

The main trade-off is that EC2 is not fully managed. The team remains responsible for operating-system updates, service management, storage review, and network protection.

### Why Amazon RDS for PostgreSQL Was Selected

The project data has a structured and relational form:

- One device produces multiple telemetry records.
- One device can receive multiple commands.
- Each command has a state such as `Pending` or `Executed`.
- The backend queries records by device ID and time.

PostgreSQL is suitable for these structured queries and command-state transitions. Amazon RDS reduces operational work compared with hosting PostgreSQL directly on EC2 and provides built-in integration with CloudWatch metrics.

The trade-off is that an RDS instance continues generating charges according to its running time and provisioned resources, even when workshop traffic is low.

### Why Amazon EBS, VPC, and an IAM Role Were Selected

- **Amazon EBS** supplies the EC2 root volume that persists the operating system, checked-out source, Python virtual environment, and local backend logs across ordinary instance restarts. Capacity, snapshots, encryption, and unattached-volume cost remain operator responsibilities.
- **Amazon VPC** provides explicit subnet, route, and Security Group boundaries. It allows the demo EC2 instance to be reachable while RDS remains private in a DB Subnet Group. This control adds configuration work and requires careful evidence.
- **AWS IAM Role** gives EC2 temporary credentials for CloudWatch publishing without storing AWS access keys in source or `.env`. The role must be limited to the required actions and resources; it does not replace network controls or the CloudWatch Agent.

### Why Amazon CloudWatch Was Selected

CloudWatch centralizes:

- EC2 `CPUUtilization`.
- Memory and disk metrics collected by CloudWatch Agent.
- FastAPI backend logs.
- RDS `CPUUtilization`.
- RDS `DatabaseConnections`.
- CloudWatch Alarm states.

This demonstrates that the system supports both functional operation and infrastructure-level monitoring.

### Services Not Used in the Current Version

| Service | Reason It Was Not Selected |
| :--- | :--- |
| **AWS Lambda** | The FastAPI backend is currently deployed as a continuously running EC2 service. Moving to Lambda would require a different deployment and database-connection model |
| **Amazon API Gateway** | The frontend and YOLO UNO currently call the EC2 REST API directly. API Gateway would introduce another service layer that is not required for the current prototype |
| **Amazon DynamoDB** | The project uses relational data structures and SQLAlchemy models designed for PostgreSQL |
| **Amazon S3** | The runtime system does not currently require object storage, file uploads, or AWS-hosted static frontend assets |
| **AWS IoT Core** | YOLO UNO currently communicates with FastAPI through HTTP REST APIs. MQTT and device certificates remain possible future improvements |
| **Amazon SQS** | The implemented command path uses PostgreSQL `Pending` rows and device polling; no SQS queue, producer, or consumer is deployed |

These services are not excluded because they are unsuitable for IoT. They are outside the current scope so that the workshop can focus on the implemented end-to-end flow between hardware, REST APIs, PostgreSQL, the dashboard, and CloudWatch.

### Cost and Simplicity Considerations

The current architecture prioritizes direct implementation and observability rather than a fully serverless design.

- **Simplicity:** EC2 can host the existing FastAPI backend without splitting it into multiple Lambda functions.
- **Managed services:** RDS reduces database-management work compared with installing PostgreSQL on EC2.
- **Cost:** EC2 and RDS generate charges based on running time and should be stopped or deleted after the workshop.
- **Learning value:** The architecture demonstrates Linux, systemd, REST APIs, PostgreSQL, IAM, Security Groups, and CloudWatch.
- **Scalability:** The current system can support a small number of devices, but larger deployments would require authentication, HTTPS, load balancing, or an event-driven architecture.

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

## Security and IAM

### Network Security Table

| Source | Destination | Port | Rule |
| :--- | :--- | :---: | :--- |
| `<ADMIN_IP>/32` | EC2 Security Group | 22 | SSH administration only |
| Demo clients | EC2 Security Group | 8000 | HTTP demo API; avoid `0.0.0.0/0` beyond the demo |
| EC2 Security Group | RDS Security Group | 5432 | PostgreSQL only |
| EC2/RDS | CloudWatch | HTTPS | Outbound monitoring path |

RDS should not be publicly accessible. Secrets stay in ignored local files; EC2 uses an IAM Role instead of hard-coded AWS keys. The current design does not claim HTTPS, authentication, HA, Multi-AZ, a load balancer, or rate limiting.

### Security and IAM Table

| Control | Current implementation | Evidence to retain | Limitation / next hardening step |
| :--- | :--- | :--- | :--- |
| Administrator access | SSH 22 restricted to `<ADMIN_IP>/32` | EC2 Security Group rule | Review/rotate key ownership; consider managed access |
| Database isolation | RDS private; 5432 accepts the EC2 Security Group only | RDS setting, subnet group, SG reference | Review NACL/routes and TLS verification |
| AWS credentials | EC2 IAM Role with `CloudWatchAgentServerPolicy` for monitoring | Instance profile and attached policy | Replace broad managed permissions with reviewed resource-scoped policy where practical |
| Application secrets | `.env` and `hardware/include/secrets.h` remain local and ignored | Redacted file locations and `git status` | Move production secrets to an approved managed secret service |
| Public API | Direct HTTP port 8000 for supervised demo | EC2 SG and health request | Add HTTPS, authentication, authorization, and rate limiting before production |
| Database identity | Dedicated PostgreSQL user in `DATABASE_URL` | Redacted connection configuration | Restrict grants, rotate password, audit access |

### Principle of Least Privilege

Grant only the actions needed by each identity, limit network sources and destinations, and avoid permanent credentials. The workshop does not require `AdministratorAccess`: the human operator needs only the approved provisioning/inspection actions, EC2 needs only monitoring publication permissions, RDS accepts PostgreSQL only from the EC2 Security Group, and the database user should receive only application-required privileges. Any temporary broad permission used during troubleshooting must be documented, time-bounded, reviewed, and removed.

## Current Operational Model

- One Amazon Linux EC2 instance runs persistent FastAPI/Uvicorn as `aws-iot-backend` under `systemd`.
- One EBS root volume holds the operating system, source, virtual environment, and local log files.
- One private RDS for PostgreSQL instance stores devices, telemetry, and command states.
- A local React/Vite dashboard and one YOLO UNO device call the EC2 REST API through periodic HTTP polling.
- CloudWatch Agent ships guest metrics and two backend log files; native EC2/RDS metrics and six documented alarms support operations.
- Deployment, scaling decisions, recovery, and clean-up are manual runbook activities in the current version.

## Future Scalability Options and Current Limitations

The `device_id` route structure and relational schema can support more rooms, but the present acceptance scope is `room_01`. A single EC2 endpoint and periodic HTTP polling are simple for a prototype but create a changing-public-IP risk, polling delay, and a single compute failure domain.

Possible future options include a stable DNS/HTTPS endpoint, authentication and per-device authorization, a load balancer with multiple stateless backend instances, Auto Scaling, managed MQTT through AWS IoT Core, queue-based processing such as SQS, caching, read replicas, Multi-AZ database deployment, containers, and Infrastructure as Code. These require a new architecture, cost/security review, implementation, and tests.

**Auto Scaling, Amazon SQS, and an event-driven architecture are not deployed in the current project.** They are future scalability options only.

## Expected result and troubleshooting

Every arrow in the architecture should map to one API call, network rule, database action, or metric/log path. If a connection is unclear, identify its source, destination, port, identity, and expected evidence before provisioning.

Next: [build the AWS foundation](../5.4-AWS-Infrastructure-Setup/).
