---
title: "Workshop Overview"
date: "2026-07-28"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Workshop Overview

## Problem and users

Room operators need one place to observe environmental conditions and control equipment remotely. Without a central system, readings are fragmented, history is difficult to inspect, and a UI click does not prove that a physical device executed the command.

The workshop serves facility operators, room managers, and learners who want practical experience with AWS, REST APIs, databases, frontend integration, and embedded IoT.

## Solution scope

The solution monitors `room_01` and supports:

- temperature, humidity, and light telemetry;
- current and historical readings;
- fan, light, and curtain control;
- a `Pending` → `Executed` command lifecycle with device ACK; and
- backend logs, infrastructure metrics, and alarms.

![Final system architecture](/images/2-Proposal/IoT_Dashboard_Architecture.png)

## Success criteria

The workshop is complete when telemetry is stored in RDS and visible on the dashboard, every supported command can be traced from creation through physical execution and ACK, and CloudWatch contains the expected logs and datapoints.

**Expected result:** The scope and acceptance criteria are clear before resources are created.

## Troubleshooting

- If the architecture image is missing, confirm the file exists under `static/images/2-Proposal/`.
- If requirements expand during implementation, record them as future improvements instead of silently changing the acceptance criteria.

Next: [prepare the required account, tools, and hardware](../5.2-Prerequisites/).
