---
title: "Cost, Security, and Clean-up"
date: "2026-07-28"
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

# Cost, Security, and Clean-up

## Cost review

Estimate before deployment with the AWS Pricing Calculator and verify actual usage with Billing or Cost Explorer when available. Include:

| Component | Main cost driver |
|---|---|
| EC2 | Instance type and running hours |
| EBS | Provisioned GB-month and snapshots |
| RDS PostgreSQL | DB class, running hours, storage, backups |
| CloudWatch | Log ingestion/storage, custom metrics, alarms |

Use small resources for the demo, but do not assume a service is free without checking current account and Region eligibility.

## Security checklist

- Restrict SSH and the backend port by source.
- Permit RDS only from the EC2 Security Group.
- Use an EC2 IAM Role and least-privilege policies.
- Store database and Wi-Fi secrets outside Git.
- Mask secrets in screenshots and logs.
- Rotate any credential that was accidentally exposed.

## Clean-up checklist

1. Export required evidence and data.
2. Terminate the EC2 instance and remove unneeded EBS volumes/snapshots.
3. Delete the RDS instance and unwanted final snapshot.
4. Delete CloudWatch alarms and workshop log groups.
5. Remove unused IAM policies/roles and Security Groups after dependencies are gone.
6. Recheck Billing/Cost Explorer.

> Stopping or leaving an RDS instance does not mean all charges have ended. Delete resources that are no longer required and verify the account afterward.

**Expected result:** The team understands the cost drivers, protects secrets, and leaves no unintended chargeable workshop resources.

Next: [summarize results and lessons learned](../5.11-Results-Challenges-Future/).
