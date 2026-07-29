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

Create the EC2 and RDS Security Groups in the same VPC. The deployed environment uses `iot-backend-sg` for backend access, `ec2-rds-1` for the EC2-to-RDS connection, and `rds-ec2-1` for the RDS-side rule.

| Security Group | Type | Source | Purpose |
| :--- | :---: | :--- | :--- |
| `iot-backend-sg` | SSH 22 | `<ADMIN_IP>/32` | Restricted administration |
| `iot-backend-sg` | Custom TCP 8000 | `0.0.0.0/0` in the current Workshop | Direct FastAPI demo access |
| `iot-backend-sg` | HTTP 80 | Rule exists in the current group | No Nginx or reverse-proxy deployment is claimed |
| `ec2-rds-1` → `rds-ec2-1` | PostgreSQL 5432 | EC2 Security Group reference | EC2-to-RDS only |

In the Workshop environment, port 8000 is exposed so that the frontend and YOLO UNO can access the FastAPI backend. A production deployment should restrict the allowed sources and use HTTPS, a reverse proxy, and authentication. The port 80 rule shown in the evidence does not prove that Nginx or another reverse proxy is running. RDS does not expose PostgreSQL port 5432 directly to `0.0.0.0/0`; it accepts the database connection from the EC2 Security Group.

The following two screenshots separate the EC2-side rules from the RDS-side Security Group relationship while redacting the administrator IP and other sensitive identifiers.

![EC2 Security Group inbound and outbound rules](/images/5-Workshop/5.4-aws-infrastructure/security-group-rules-EC2.png)
*Figure 7a. EC2 Security Group rules for `iot-backend-server`: SSH is restricted to the administrator `/32`, Workshop ports 80 and 8000 are shown, and PostgreSQL traffic on port 5432 is directed to the RDS Security Group.*

![RDS Security Group relationship with the EC2 Security Group](/images/5-Workshop/5.4-aws-infrastructure/security-group-rules-DB.png)
*Figure 7b. The RDS connectivity view shows `rds-ec2-1` as the database VPC Security Group and `ec2-rds-1` as the associated EC2 Security Group, confirming the Security Group-to-Security Group relationship.*

## Step 4 - Create the EC2 IAM Role

1. Open **IAM → Roles → Create role**.
2. Select trusted entity **AWS service → EC2**.
3. Attach `CloudWatchAgentServerPolicy` only if CloudWatch Agent will publish metrics/logs.
4. Use the deployed role name `iot-dashboard-cloudwatch-role` and create an instance profile.

Do not create long-lived access keys. The deployed role uses the AWS-managed `CloudWatchAgentServerPolicy`, allowing CloudWatch Agent to publish the required logs and metrics without hard-coded AWS access keys. The role grants permission; CloudWatch Agent is installed separately in section 5.9. Review and narrow IAM permissions for a production deployment instead of assuming the managed policy is perfectly least-privileged.

The EC2 Security page and IAM Role details confirm the role attachment and the AWS-managed policy.

![IAM Role and CloudWatchAgentServerPolicy attached to EC2](/images/5-Workshop/5.4-aws-infrastructure/ec2-iam-role-cloudwatch.png)
*Figure 5. The EC2 instance is attached to the `iot-dashboard-cloudwatch-role`, which uses `CloudWatchAgentServerPolicy` to allow CloudWatch Agent to publish logs and metrics without hard-coded AWS access keys.*

## Step 5 - Launch EC2 and configure EBS

1. Launch the approved Linux AMI as `iot-backend-server` using the deployed `t3.micro` instance type.
2. Place it in the public subnet and enable a public IPv4 address for the demo.
3. Attach the deployed EC2 Security Groups, the key pair, and the IAM instance profile.
4. Configure the EBS root volume as `gp3` with 10 GiB of capacity.
5. Add tags for project, owner, environment, and clean-up date.
6. Wait for the EC2 instance to reach **Running** and pass the displayed `3/3` status checks.

The root EBS volume is currently **In-use**, with normal I/O. The current EBS volume is not encrypted. A production deployment should enable EBS encryption with AWS KMS to protect data at rest.

Record `<EC2_PUBLIC_IP>` without publishing the private key, instance ID, or key-pair details. A changed public IP after stop/start is a known limitation; Elastic IP is only a future option.

The EC2 Instances page confirms the deployed compute size, Availability Zone, running state, and health checks.

![Amazon EC2 instance hosting the FastAPI backend](/images/5-Workshop/5.4-aws-infrastructure/ec2-instance-running.png)
*Figure 4. The `iot-backend-server` Amazon EC2 instance hosting the FastAPI backend in the Running state and passing all status checks.*

## Step 6 - Create Amazon RDS for PostgreSQL

1. Open **RDS → Databases → Create database** and select PostgreSQL.
2. Choose the deployed `db.t4g.micro` class and the approved storage configuration.
3. Set the initial database name to `iot_dashboard`.
4. Select the project VPC, DB Subnet Group `rds-ec2-db-subnet-group-1`, and the deployed RDS Security Groups.
5. Keep the Internet access gateway disabled, as shown in the deployed RDS configuration.
6. Store the master password securely as `<DB_PASSWORD>`; do not add it to screenshots or Git.
7. Enable only the backup, encryption, and monitoring options approved for the project.
8. Wait for `iot-dashboard-db` to reach **Available** in `ap-southeast-1c` and record `<RDS_ENDPOINT>` privately.

Do not claim Multi-AZ, read replicas, High Availability, RDS Proxy, a public endpoint, or enabled IAM database authentication because the current evidence does not prove those features.

The RDS summary and Connectivity & security page confirm the PostgreSQL engine, instance class, DB Subnet Group, Availability Zone, and disabled Internet access gateway.

![Amazon RDS PostgreSQL instance using a DB Subnet Group](/images/5-Workshop/5.4-aws-infrastructure/rds-postgresql-available.png)
*Figure 6. The `iot-dashboard-db` Amazon RDS for PostgreSQL instance in the Available state, using the `rds-ec2-db-subnet-group-1` DB Subnet Group with the Internet access gateway disabled.*

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

The figures above provide redacted evidence of the EC2 **Running** state, attached IAM Role, RDS **Available** state, DB Subnet Group, and both Security Group rule sets. Preserve separate command output for the successful EC2-to-RDS port test and EBS volume details.

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
