---
title: "Cost, Security and Clean-up"
date: "2026-07-28"
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

## Overview and objectives

Understand the cost drivers, review the prototype security boundary, preserve required evidence, and remove resources in dependency order. Exact prices are not stated because they depend on region, resource sizes, retention, transfer, and current pricing.

## Cost review

| Resource | Cost driver | Workshop control |
| :--- | :--- | :--- |
| Amazon EC2 | Instance type and running hours | Small approved instance; stop/terminate after use |
| Amazon EBS | Provisioned type/capacity and snapshots | Right-size and remove unattached volumes/snapshots |
| Amazon RDS for PostgreSQL | DB class, running time, storage, backups | Small workshop DB; delete after evidence if approved |
| Amazon CloudWatch | Custom metrics, log ingestion/storage, alarms | 60-second guest metrics where needed; short retention |
| Data transfer | Traffic between users/device and EC2 | Reasonable telemetry and polling intervals |

Use the AWS Pricing Calculator or the actual bill for a dated estimate. Do not copy an unverified price into the report.

## Security review

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

## Pre-clean-up evidence

Before deleting anything, preserve:

1. architecture and resource inventory;
2. EC2/service health and deployment commit;
3. RDS table/query evidence without credentials;
4. telemetry, command, and ACK test records;
5. CloudWatch log/metric/alarm screenshots; and
6. hardware/frontend demonstration evidence.

Confirm who owns snapshots and how long evidence must be retained.

## Clean-up procedure

1. Stop new telemetry/command traffic and place actuators in a safe state.
2. Stop or terminate EC2 according to the handover decision.
3. Delete unattached EBS volumes and unnecessary snapshots after confirming ownership.
4. Delete RDS, taking a final snapshot only when retention is required and approved.
5. Delete CloudWatch alarms.
6. Delete unneeded log groups and custom monitoring data according to retention policy.
7. Remove the project IAM Role/instance profile if it is used only by this project.
8. Delete Security Groups only after dependent ENIs/resources are gone.
9. Remove workshop-only route/network resources only when no shared dependency exists.
10. Open Billing/Cost Explorer and verify that no unexpected project resource remains.

Stopping RDS is temporary and subject to service limits; it may start automatically again. Deleting the database and other billable resources is the way to avoid continuing long-term charges, subject to the team's retention decision.

## Verification and troubleshooting

- Re-run the tagged-resource inventory in the correct region.
- Check for stopped instances, unattached EBS volumes, retained RDS snapshots, log groups, and idle alarms.
- If a Security Group cannot be deleted, find the dependent ENI/resource rather than forcing deletion.
- If a database cannot be deleted, review deletion protection and snapshot requirements with the owner.
- Record each deleted resource ID in the clean-up evidence; never expose credentials.

<!-- TODO IMAGE: /images/5-Workshop/5.10-cleanup/aws-resource-inventory.png — Redacted before/after AWS resource inventory and Billing/Cost Explorer check showing the workshop clean-up result. -->

**Expected result:** required evidence is retained, no unapproved billable workshop resource remains, and the security review records current limitations without claiming production readiness.

Next: [document results, challenges, and future improvements](../5.11-Results-Challenges-Future/).
