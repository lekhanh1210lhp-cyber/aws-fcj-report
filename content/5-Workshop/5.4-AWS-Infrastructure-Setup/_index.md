---
title: "AWS Infrastructure Setup"
date: "2026-07-28"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Overview and objectives

Create the network, identity, compute, storage, and database foundation for the prototype. Use placeholders in notes and screenshots; never publish an account ID, password, private endpoint, or key.

## Step 1 - Select the region and address plan

In the AWS Console, select the confirmed project region. This workshop uses **Asia Pacific (Singapore), `ap-southeast-1`**, as its baseline. Record non-overlapping CIDRs for the VPC, one public subnet, and at least two DB subnets in different Availability Zones.

**Expected result:** every resource created below appears in the same region and uses the agreed name prefix.

## Step 2 - Create or select the VPC and subnets

1. Open **VPC → Your VPCs** and create/select the project VPC.
2. Enable DNS resolution and DNS hostnames.
3. Create/select a public subnet for EC2.
4. Attach an Internet Gateway to the VPC.
5. Add `0.0.0.0/0 → Internet Gateway` to the public subnet route table.
6. Create/select two database subnets in separate Availability Zones. Do not add an Internet Gateway route for private DB subnets.
7. In **RDS → Subnet groups**, create a DB Subnet Group containing both DB subnets.

**Expected result:** EC2 can receive a public IPv4 address, while the database subnets remain private.

## Step 3 - Create Security Groups

Create `iot-ec2-sg` and `iot-rds-sg` in the same VPC.

| Security Group | Type | Source | Purpose |
| :--- | :---: | :--- | :--- |
| `iot-ec2-sg` | SSH 22 | `<ADMIN_IP>/32` | Restricted administration |
| `iot-ec2-sg` | Custom TCP 8000 | Approved demo clients | Direct Uvicorn demo access |
| `iot-ec2-sg` | HTTP 80 | Only when a reverse proxy exists | Not required for direct port 8000 |
| `iot-rds-sg` | PostgreSQL 5432 | **`iot-ec2-sg`**, not an IP CIDR | EC2-to-RDS only |

Using `0.0.0.0/0` on port 8000 may be temporarily acceptable for a supervised demo but is not a production recommendation. Do not make RDS publicly accessible.

## Step 4 - Create the EC2 IAM Role

1. Open **IAM → Roles → Create role**.
2. Select trusted entity **AWS service → EC2**.
3. Attach `CloudWatchAgentServerPolicy` only if CloudWatch Agent will publish metrics/logs.
4. Use the source runbook name `iot-dashboard-cloudwatch-role` (or the project-approved equivalent) and create an instance profile.

Do not create long-lived access keys. The role grants permission; CloudWatch Agent is installed separately in section 5.9.

## Step 5 - Launch EC2 and configure EBS

1. Launch the approved Linux AMI and a small workshop instance type.
2. Place it in the public subnet and enable a public IPv4 address for the demo.
3. Attach `iot-ec2-sg`, the key pair, and the IAM instance profile.
4. Configure the EBS root volume with the approved type, capacity, and encryption setting.
5. Add tags for project, owner, environment, and clean-up date.
6. Wait for both EC2 status checks to pass.

Record `<EC2_PUBLIC_IP>` without publishing the private key. A changed public IP after stop/start is a known limitation; Elastic IP is only a future option.

## Step 6 - Create Amazon RDS for PostgreSQL

1. Open **RDS → Databases → Create database** and select PostgreSQL.
2. Choose the workshop-sized instance and storage configuration.
3. Set the initial database name to `iot_dashboard`.
4. Select the project VPC, DB Subnet Group, and `iot-rds-sg`.
5. Set **Public access: No**.
6. Store the master password securely as `<DB_PASSWORD>`; do not add it to screenshots or Git.
7. Enable only the backup, encryption, and monitoring options approved for the project.
8. Wait for status **Available** and record `<RDS_ENDPOINT>`.

Do not claim Multi-AZ unless the deployed RDS configuration proves it.

## Step 7 - Verify access and network

Connect from Windows PowerShell:

```powershell
ssh -i "$env:USERPROFILE\.ssh\<KEY_FILE>.pem" <EC2_USER>@<EC2_PUBLIC_IP>
```

From EC2 Linux Bash, verify DNS and TCP reachability:

```bash
getent hosts <RDS_ENDPOINT>
nc -vz <RDS_ENDPOINT> 5432
```

If `nc` is unavailable, install the appropriate netcat package for the selected Linux distribution. A successful TCP test proves the route and Security Groups, not database credentials.

## Expected Result and Evidence

Capture redacted evidence of the EC2 **running** state, attached EBS volume and IAM Role, RDS **available** state, DB Subnet Group, both Security Group rule sets, and successful EC2-to-RDS port test.

<!-- TODO IMAGE: /images/5-Workshop/5.4-aws-infrastructure/ec2-instance-running.png — EC2 console showing the workshop instance in Running state; redact account ID, public DNS/IP when required, and key-pair details. -->
<!-- TODO IMAGE: /images/5-Workshop/5.4-aws-infrastructure/ec2-iam-role-cloudwatch.png — EC2 instance profile showing iot-dashboard-cloudwatch-role and CloudWatchAgentServerPolicy; redact account ID and role ARN. -->
<!-- TODO IMAGE: /images/5-Workshop/5.4-aws-infrastructure/rds-postgresql-available.png — Private RDS for PostgreSQL instance in Available state with DB Subnet Group; redact endpoint and account identifiers. -->
<!-- TODO IMAGE: /images/5-Workshop/5.4-aws-infrastructure/security-group-rules.png — EC2 and RDS Security Group rules showing SSH restriction, demo port 8000, and PostgreSQL 5432 sourced from the EC2 Security Group; redact public IPs. -->

## Troubleshooting

| Symptom | Check |
| :--- | :--- |
| SSH timeout | Public IP, route table, Internet Gateway, `<ADMIN_IP>/32`, local firewall |
| SSH permission denied | Login user and local key permissions; never change the remote SG to solve a key error |
| RDS timeout | Endpoint/region, DB subnet routing, RDS SG source, network ACL |
| RDS connection refused | Correct port and database status |
| EC2 cannot publish metrics | IAM instance profile attachment and outbound HTTPS |
| Browser cannot reach port 8000 | Backend bind address, service state, EC2 SG, client network |

Next: [deploy FastAPI and connect PostgreSQL](../5.5-Backend-and-Database/).
