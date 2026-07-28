---
title: "Monitoring with CloudWatch"
date: "2026-07-28"
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

# Monitoring with CloudWatch

## Step 1 - Install and configure the agent

Install Amazon CloudWatch Agent on EC2. Configure it to collect the project’s backend output and error log files, plus memory and disk metrics. Start and enable the agent, then verify its status.

Use separate log streams or clear file paths for:

- backend application events; and
- backend errors.

Do not put database credentials or request secrets in logs.

## Step 2 - Review metrics

Confirm datapoints for:

- EC2 `CPUUtilization`;
- memory and disk metrics in the `CWAgent` namespace;
- RDS `CPUUtilization`; and
- RDS `DatabaseConnections`.

## Step 3 - Create alarms

Create five alarms that cover EC2 CPU, EC2 memory, EC2 disk, RDS CPU, and RDS connections. Select thresholds suitable for the small demo instance and document the evaluation period.

Interpret states correctly:

- **OK:** the condition is not breached;
- **In alarm:** the configured condition is breached;
- **Insufficient data:** CloudWatch lacks enough recent datapoints.

SNS/email notification is outside this workshop’s scope.

**Expected result:** Logs arrive, metrics contain current datapoints, and five alarms are visible with documented thresholds.

## Troubleshooting

- If logs are absent, check file paths, read permissions, agent configuration, IAM Role, and agent status.
- `Insufficient data` immediately after creation may be normal; wait for the configured evaluation periods and confirm the metric dimensions.

Next: [review cost, security, and clean-up](../5.10-Cost-Security-Cleanup/).
