---
title: "Architecture and Service Design"
date: "2026-07-28"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Architecture and Service Design

The React client and YOLO UNO are outside AWS. FastAPI runs on EC2 in a public subnet; PostgreSQL runs in RDS through a DB Subnet Group. The EC2 instance uses an IAM Role to publish operational data to CloudWatch without hard-coded AWS access keys.

![AWS IoT Dashboard architecture](/images/2-Proposal/IoT_Dashboard_Architecture.png)

## Data flows

1. **Telemetry:** YOLO UNO → `POST /api/telemetry` → FastAPI → RDS → dashboard latest/history views.
2. **Command:** dashboard → create command → RDS row with `Pending` status.
3. **Execution and ACK:** YOLO UNO polls the latest command, operates the actuator, then posts ACK; FastAPI updates the row to `Executed`.
4. **Observability:** backend log files → CloudWatch Agent → CloudWatch Logs; EC2/RDS/CWAgent metrics → CloudWatch alarms.

## Service decisions and security boundaries

- **EC2:** direct control over FastAPI, Uvicorn, `systemd`, and log files.
- **RDS for PostgreSQL:** managed relational storage and persistent telemetry/command history.
- **EBS:** operating system, application, and local log storage for EC2.
- **CloudWatch:** a single location for logs, metrics, and alarms.
- **IAM Role:** temporary AWS credentials for the EC2 workload.

Restrict SSH to the team’s public IP, allow the backend port only from approved sources, and configure the RDS Security Group to accept PostgreSQL (`5432`) only from the EC2 Security Group.

**Expected result:** Every component, trust boundary, and data flow has an explicit purpose.

## Troubleshooting

- If the device cannot reach EC2, check the public IP, route, Network ACL, and EC2 Security Group.
- If EC2 cannot reach RDS, verify the RDS endpoint, port, DB Subnet Group, and source Security Group rule.

Next: [provision the AWS infrastructure](../5.4-AWS-Infrastructure-Setup/).
