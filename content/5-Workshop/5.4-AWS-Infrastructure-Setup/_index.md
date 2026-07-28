---
title: "AWS Infrastructure Setup"
date: "2026-07-28"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# AWS Infrastructure Setup

## Step 1 - Network and Security Groups

In `ap-southeast-1`, create or select a VPC with a public subnet for EC2 and at least two suitable subnets for the RDS DB Subnet Group.

Create:

- `iot-ec2-sg`: SSH (`22`) from your public IP and the backend port (`8000`) only from approved client IPs.
- `iot-rds-sg`: PostgreSQL (`5432`) with `iot-ec2-sg` as the source.

Do not expose RDS to `0.0.0.0/0`.

## Step 2 - IAM Role and EC2

Create an EC2 IAM Role, attach `CloudWatchAgentServerPolicy`, and launch a small Linux instance in the public subnet. Attach `iot-ec2-sg`, the IAM Role, and an EBS volume sized for the demo.

## Step 3 - RDS PostgreSQL

Create a PostgreSQL instance with:

- database name: `iot_dashboard`;
- private connectivity;
- the DB Subnet Group; and
- `iot-rds-sg`.

From EC2, install the PostgreSQL client and test:

```bash
psql "host=<RDS_ENDPOINT> port=5432 dbname=iot_dashboard user=<DB_USER> sslmode=require"
```

**Expected result:** `psql` opens a PostgreSQL session from EC2 while direct public database access remains blocked.

## Troubleshooting

- A timeout usually indicates routing or Security Group configuration; an authentication error indicates the endpoint, user, password, or database name.
- If EC2 lacks CloudWatch permissions, confirm that the IAM Role is attached to the instance, not only created.

Next: [deploy the backend and connect the database](../5.5-Backend-and-Database/).
