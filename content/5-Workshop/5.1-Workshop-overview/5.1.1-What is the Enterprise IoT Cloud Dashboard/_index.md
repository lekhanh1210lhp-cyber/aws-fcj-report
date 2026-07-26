---
title: "What is the Enterprise IoT Cloud Dashboard"
date: "2026-06-15"
weight: 1
chapter: false
pre: " <b> 5.1.1 </b> "
---

#### Brief Definition

**The Enterprise IoT Cloud Dashboard** is a system built on AWS designed for centralized smart building management (BMS). 

In essence, the architecture utilizes 5 core layers to operate smoothly:
- **Frontend**: Built with React.
- **Backend**: Built with FastAPI and hosted on AWS EC2.
- **Database**: Managed relational database using PostgreSQL RDS.
- **Monitoring**: System tracking and error logging via CloudWatch.
- **IoT Devices**: Edge devices simulated by a Python Simulator.

The goal of this system is to securely process sensor data and enable reliable, bi-directional communication between remote dashboards and hardware edge devices like YOLO Uno or ESP32.

#### Why is this system needed?

Traditional building management lacks centralized cloud integration. This dashboard solves this by providing 3 main capabilities:

- **Centralized Management**: Enables centralized smart building management (BMS) through a highly available AWS cloud infrastructure.
- **Real-time Monitoring**: Facilitates telemetry data ingestion, allowing the system to continuously receive temperature, humidity, light, and device status.
- **Two-Way Communication**: Allows administrators to execute remote commands from the dashboard directly to the edge devices, such as turning fans ON/OFF or opening curtains.

#### Operational Architecture

The process of handling telemetry data and remote commands proceeds as follows:

| Step  | Name                          | Action Description                                                                                                            |
| :---- | :---------------------------- | :---------------------------------------------------------------------------------------------------------------------------- |
| **1** | **Data Generation** (Tạo dữ liệu)     | The Python Simulator generates randomized, realistic sensor data and uses HTTP POST requests to send it to the FastAPI backend. |
| **2** | **Processing & Storage** (Xử lý & Lưu trữ) | The FastAPI backend validates the incoming JSON data payloads using Pydantic and securely stores the data in the PostgreSQL RDS.                                                   |
| **3** | **Visualization & Control** (Trực quan hóa & Điều khiển)     | The React dashboard fetches the stored telemetry data for display and dispatches remote commands (queued in PostgreSQL) back to the polling IoT devices.                            |

![IoT Architecture](/images/5-Workshop/5.1-Workshop-overview/iot-architecture-01.jpg)