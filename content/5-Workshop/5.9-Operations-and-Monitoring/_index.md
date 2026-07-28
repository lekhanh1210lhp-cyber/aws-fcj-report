---
title: "Operations & Monitoring"
date: "2026-06-15"
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

#### Objectives

This section focuses on keeping the IoT dashboard healthy after deployment. The goal is to ensure that the team can monitor system activity, detect abnormal behavior, and respond quickly to incidents.

We will cover:

1. Monitoring backend health with AWS CloudWatch.
2. Reviewing EC2 metrics such as CPU, memory, and network usage.
3. Logging and auditing command execution for maintainability and security.

#### Monitoring workflow

1. Open the AWS Console and navigate to **CloudWatch**.
2. Review the log groups related to the FastAPI application and EC2 instance.
3. Check alarms and dashboards for CPU spikes, API errors, and service availability.
4. Confirm that command execution and telemetry ingestion are visible in logs.

<!-- Insert screenshot: CloudWatch dashboard showing metrics and alarms -->
> Placeholder for screenshot: AWS CloudWatch monitoring dashboard.

#### Expected result

By the end of this section, the team should be able to observe the health of the platform and identify operational issues before they affect users.
