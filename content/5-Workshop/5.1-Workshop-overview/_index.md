---
title: "Introduction"
date: "2026-06-15"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### Overview

In this project, we will focus on building an Enterprise IoT Cloud Dashboard on AWS for centralized smart building management (BMS).

The main objective is to establish a 5-layer architecture utilizing React (Frontend), FastAPI on AWS EC2 (Backend), PostgreSQL RDS (Database), CloudWatch (Monitoring), and a Python Simulator (IoT Devices). The workflow consists of the following steps:

1.  **Infrastructure & Database:** Provisioning foundational AWS resources (VPC, EC2) and deploying a PostgreSQL RDS instance.
2.  **API & IoT Simulation:** Building telemetry ingestion API endpoints and developing a Python script to simulate YOLO Uno/ESP32 edge devices.
3.  **Frontend & Integration:** Building a React dashboard for historical data visualization and remote device control, ensuring full bi-directional communication.

> 💡 **Highlight:** This solution allows administrators to **view trends and send commands directly from the UI**, ensuring flawless data flow from the Simulator to the Dashboard and back.

![overview](/images/5-Workshop/5.1-Workshop-overview/overview_diagram.png)

#### Implementation Steps

1. [System Architecture & Cloud Infrastructure Foundation](5.1.1-Architecture/)
2. [Database Design & REST API Implementation](5.1.2-Backend/)