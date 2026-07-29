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

## Step 4 - Clean up only project-owned resources

1. Stop new telemetry/command traffic and place actuators in a safe state.
2. Preserve the evidence and final-snapshot decision recorded in Step 3.
3. Stop and terminate the project EC2 instance after handover approval.
4. Verify the EC2 root EBS `DeleteOnTermination` setting; delete only project-owned unattached volumes and snapshots that were actually created.
5. Delete the project RDS for PostgreSQL instance; create a final snapshot only when retention is required and approved.
6. Delete the six project CloudWatch alarms and, after the retention decision, the two project log groups.
7. Remove `iot-dashboard-cloudwatch-role` and its instance profile only if no other workload uses them.
8. Delete `iot-ec2-sg` and `iot-rds-sg` after their ENI/resource dependencies are gone.
9. If this Workshop created a dedicated DB Subnet Group, public subnet, two DB subnets, route table, Internet Gateway, and VPC, delete them in dependency order. Do not delete selected/shared network resources.
10. Open Billing/Cost Explorer and the tagged-resource inventory to verify that no unexpected project resource remains.

The implementation creates no S3 bucket and no CloudFormation, SAM, CDK, or Terraform stack/state. Therefore, clean-up does **not** instruct participants to delete a bucket or stack. If a future deployment adds one, update the inventory and dependency order first.

Stopping RDS is temporary and subject to service limits; it may start automatically again. Deleting the database and other billable resources is the way to avoid continuing long-term charges, subject to the team's retention decision.

## Step 5 - Verify the clean-up

- Re-run the tagged-resource inventory in the correct region.
- Check for stopped instances, unattached EBS volumes, retained RDS snapshots, log groups, and idle alarms.
- If a Security Group cannot be deleted, find the dependent ENI/resource rather than forcing deletion.
- If a database cannot be deleted, review deletion protection and snapshot requirements with the owner.
- Record each deleted resource ID in the clean-up evidence; never expose credentials.

<!-- TODO IMAGE: /images/5-Workshop/5.10-cleanup/aws-resource-inventory.png — Redacted before/after AWS resource inventory and Billing/Cost Explorer check showing the workshop clean-up result. -->

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

Next: [document results, challenges, and future improvements](../5.11-Results-Challenges-Future/).
