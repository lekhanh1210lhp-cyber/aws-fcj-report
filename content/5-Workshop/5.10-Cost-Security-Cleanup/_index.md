---
title: "Cost, Security and Clean-up"
date: "2026-07-28"
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

## Overview and objectives

Understand the cost drivers, review the prototype security boundary, preserve required evidence, and remove resources in dependency order. Exact prices are not stated because they depend on region, resource sizes, retention, transfer, and current pricing.

## Step 1 - Review cost drivers

| Resource | Cost driver | Workshop control |
| :--- | :--- | :--- |
| Amazon EC2 | Instance type and running hours | Small approved instance; stop/terminate after use |
| Amazon EBS | Provisioned type/capacity and snapshots | Right-size and remove unattached volumes/snapshots |
| Amazon RDS for PostgreSQL | DB class, running time, storage, backups | Small workshop DB; delete after evidence if approved |
| Amazon CloudWatch | Custom metrics, log ingestion/storage, alarms | 60-second guest metrics where needed; short retention |
| Data transfer | Traffic between users/device and EC2 | Reasonable telemetry and polling intervals |

Use the AWS Pricing Calculator or the actual bill for a dated estimate. Do not copy an unverified price into the report.

## Step 2 - Review the security boundary

- Use least-privilege identities and MFA.
- Use an EC2 IAM Role; do not hard-code AWS access keys.
- Restrict SSH port 22 to `<ADMIN_IP>/32`.
- Allow RDS 5432 only from the EC2 Security Group.
- Keep RDS private.
- Ignore `.env`, `.pem`, `.key`, and `hardware/include/secrets.h`.
- Use placeholders in documentation and redact screenshots.
- Treat public HTTP port 8000 as a demo limitation.
- Review outbound rules, log retention, database users, and resource tags.
- Rotate any secret immediately if it appears in Git, terminal history, a screenshot, or a demo video.

Production recommendations include a reverse proxy, HTTPS, authentication, authorization, managed secrets, backups, and a reviewed network design. They are not current implemented features.

## Step 3 - Preserve pre-clean-up evidence

Before deleting anything, preserve:

1. architecture and resource inventory;
2. EC2/service health and deployment commit;
3. RDS table/query evidence without credentials;
4. telemetry, command, and ACK test records;
5. CloudWatch log/metric/alarm screenshots; and
6. hardware/frontend demonstration evidence.

Confirm who owns snapshots and how long evidence must be retained.

## AWS Resource Inventory Before Clean-up

The following table summarizes the resources currently used by the project. Required screenshots, logs, validation results, source code, and demonstration videos must be preserved before any resource is stopped or deleted.

| Resource | Name or Role | Current Status | Evidence | Clean-up Action |
| :--- | :--- | :--- | :--- | :--- |
| Amazon EC2 | `iot-backend-server` | Running, `t3.micro`, passing `3/3` status checks | EC2 Instances page | Preserve required source code, configuration, and logs; then stop or terminate the instance |
| Amazon EBS | Root volume of `iot-backend-server` | `gp3`, 10 GiB, In use, status Okay, normal I/O, not encrypted | EC2 → Volumes page | Verify `Delete on termination`; create a snapshot if required, then remove the volume when no longer used |
| Amazon RDS for PostgreSQL | `iot-dashboard-db` | Available, PostgreSQL, `db.t4g.micro`, Internet access gateway disabled | RDS Database page | Create a final snapshot if required, then delete the DB instance |
| DB Subnet Group | `rds-ec2-db-subnet-group-1` | In use by RDS | RDS Connectivity section | Delete only after the DB instance and dependent resources have been removed |
| EC2 Security Groups | `iot-backend-sg`, `ec2-rds-1` | In use; controlling SSH, backend access, and RDS connectivity | EC2 Security tab | Delete after EC2 and the related network interfaces no longer use them |
| RDS Security Group | `rds-ec2-1` | In use; allowing access from the EC2 Security Group | RDS Security Group rules | Delete after RDS and dependent network interfaces have been removed |
| IAM Role | `iot-dashboard-cloudwatch-role` | Attached to EC2 | EC2 Security and IAM Role pages | Detach it from EC2, verify dependencies, and then delete the role |
| IAM Policy | `CloudWatchAgentServerPolicy` | AWS-managed policy attached to the IAM Role | IAM Permissions | Do not delete the AWS-managed policy; remove or delete only the project IAM Role |
| CloudWatch Log Group | `/aws/ec2/aws-iot-dashboard/backend` | Active, containing FastAPI access logs and HTTP status codes | CloudWatch Logs | Preserve required logs, then configure retention or delete the log group |
| CloudWatch Dashboard | `ec2-rds-metrics` | Active, displaying EC2 and RDS metrics | CloudWatch Dashboard | Preserve metric evidence, then delete the dashboard when no longer required |
| CloudWatch Alarms | Five EC2 and RDS alarms | Some `OK`, some `Insufficient data`; no notification actions attached | CloudWatch Alarms | Preserve configuration evidence, then delete all alarms |
| FastAPI backend service | `aws-iot-backend` on EC2 | Serving REST APIs and producing access logs | EC2 and CloudWatch Logs | Back up source code, service files, and environment configuration before terminating EC2 |
| Firmware artifact | `firmware.bin` | Successfully built with PlatformIO using the `yolo_uno` environment | Terminal showing `SUCCESS` | Preserve locally or as a repository artifact; it is not an AWS resource to delete |

> **Note:** Do not begin clean-up until all required screenshots, logs, validation results, source code, and demonstration videos have been preserved. The current EBS volume is not encrypted; a production deployment should use EBS encryption with AWS KMS.

<!-- TODO: Capture this rendered table as aws-resource-inventory.png -->

## Step 4 - Clean up only project-owned resources

1. Preserve screenshots, logs, source code, test results, and demonstration videos; stop new telemetry/command traffic and place actuators in a safe state.
2. Back up the required backend source, service files, and environment configuration.
3. Decide whether retention requires an approved final RDS snapshot and create it if needed.
4. Stop or terminate `iot-backend-server` after handover approval.
5. Verify the root EBS volume's `DeleteOnTermination` setting; remove only project-owned volumes or snapshots that are no longer required.
6. Delete `iot-dashboard-db` when the database and its retained data are no longer required.
7. Delete the five project CloudWatch alarms documented in Section 5.9.
8. Delete the `ec2-rds-metrics` CloudWatch dashboard after preserving its evidence.
9. Configure retention for or delete `/aws/ec2/aws-iot-dashboard/backend` after preserving required logs.
10. Detach and delete `iot-dashboard-cloudwatch-role` and its instance profile only if no other workload uses them. Do not delete the AWS-managed `CloudWatchAgentServerPolicy`.
11. Delete `iot-backend-sg`, `ec2-rds-1`, and `rds-ec2-1` only after their EC2, RDS, ENI, and Security Group dependencies are gone.
12. Delete `rds-ec2-db-subnet-group-1` only after RDS no longer uses it. Do not delete selected or shared VPC resources.
13. Recheck EC2, EBS, RDS, CloudWatch, Billing/Cost Explorer, and the tagged-resource inventory for unexpected project-owned resources.

The implementation creates no S3 bucket and no CloudFormation, SAM, CDK, or Terraform stack/state. Therefore, clean-up does **not** instruct participants to delete a bucket or stack. If a future deployment adds one, update the inventory and dependency order first.

Stopping RDS is temporary and subject to service limits; it may start automatically again. Deleting the database and other billable resources is the way to avoid continuing long-term charges, subject to the team's retention decision.

## Step 5 - Verify the clean-up

- Re-run the tagged-resource inventory in the correct region.
- Check for stopped instances, unattached EBS volumes, retained RDS snapshots, log groups, and idle alarms.
- If a Security Group cannot be deleted, find the dependent ENI/resource rather than forcing deletion.
- If a database cannot be deleted, review deletion protection and snapshot requirements with the owner.
- Record each deleted resource ID in the clean-up evidence; never expose credentials.

## Expected Result

Required evidence is retained, no unapproved billable project-owned Workshop resource remains, shared resources are untouched, and the security review records current limitations without claiming production readiness.

## Troubleshooting

| Symptom | Check |
| :--- | :--- |
| Security Group cannot be deleted | Find dependent ENIs, EC2, RDS, or referenced Security Groups |
| VPC/subnet cannot be deleted | Check Internet Gateway, route table associations, DB Subnet Group, and ENIs |
| RDS deletion is blocked | Deletion protection, final snapshot name, retained automated backups, and owner approval |
| EBS cost remains | Unattached volumes and snapshots actually owned by this project |
| CloudWatch cost remains | Log-group retention/ingestion and alarms; confirm the agent stopped with EC2 |
| Resource ownership is uncertain | Stop deletion, use tags/inventory and obtain owner confirmation |

Next: [document results, challenges, and future improvements](../5.11-results-challenges-future/).
