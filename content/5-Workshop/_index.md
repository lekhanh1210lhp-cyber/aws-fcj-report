---
title: "AWS IoT Dashboard Workshop"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# AWS IoT Monitoring and Control Dashboard

This workshop builds an end-to-end environmental monitoring and device-control system for `room_01`. A YOLO UNO reads temperature, humidity, and light data; a FastAPI service on Amazon EC2 stores telemetry in Amazon RDS for PostgreSQL; a React dashboard displays current and historical data; and Amazon CloudWatch provides logs, metrics, and alarms.

The control flow is deliberately verifiable: the dashboard creates a command with `Pending` status, the device polls and executes it, then sends an acknowledgement (ACK) that changes the status to `Executed`.

## Learning outcomes

After completing the workshop, you can:

- deploy a FastAPI backend on EC2 and connect it securely to RDS;
- send telemetry from YOLO UNO and persist it in PostgreSQL;
- control a fan, light, and curtain from a React dashboard;
- validate the complete command and ACK lifecycle;
- collect operational evidence with CloudWatch; and
- estimate cost, apply basic security controls, and remove chargeable resources.

## Workshop contents

1. [Workshop Overview](5.1-Workshop-Overview/)
2. [Prerequisites](5.2-Prerequisites/)
3. [Architecture and Service Design](5.3-Architecture-and-Service-Design/)
4. [AWS Infrastructure Setup](5.4-AWS-Infrastructure-Setup/)
5. [Backend Deployment and Database Integration](5.5-Backend-and-Database/)
6. [Hardware Integration](5.6-Hardware-Integration/)
7. [Frontend Integration](5.7-Frontend-Integration/)
8. [End-to-End Testing and Validation](5.8-End-to-End-Testing/)
9. [Monitoring with CloudWatch](5.9-CloudWatch-Monitoring/)
10. [Cost, Security, and Clean-up](5.10-Cost-Security-Cleanup/)
11. [Results, Challenges, and Future Improvements](5.11-Results-Challenges-Future/)
12. [Project Handover](5.12-Project-Handover/)

> Use the Singapore Region (`ap-southeast-1`) consistently. Replace all values written as `<PLACEHOLDER>` with your own values and never publish passwords, tokens, Wi-Fi credentials, or private keys.
