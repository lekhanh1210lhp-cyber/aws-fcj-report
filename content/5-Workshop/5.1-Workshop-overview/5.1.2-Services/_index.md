---
title: "Services & Technologies"
date: "2026-06-15"
weight: 2
chapter: false
pre: " <b> 5.1.2 </b> "
---

The solution architecture is built upon the coordination of the following key components and services:

#### React (Frontend Dashboard)

A functional, responsive web application that serves as the user interface for centralized smart building management.

- **Data Visualization:** Fetches and renders real-time telemetry (temperature, humidity, light) into UI cards and plots historical data using Chart.js/Recharts.
- **Control Panel:** Builds toggle switches in the UI, allowing administrators to dispatch remote commands for fans, lights, and curtains.
- **Multi-building Navigation:** Implements a sidebar layout to navigate between different building locations like Hanoi, Da Nang, and HCM.

#### FastAPI on AWS EC2 (Backend)

The core backend server initialized with FastAPI and actively running on an Ubuntu EC2 instance.

- **Data Ingestion & Validation:** Receives telemetry data and implements Pydantic validators to reject malformed JSON data payloads.
- **Command Dispatching:** Provides API endpoints for the dashboard to send commands and for IoT devices to retrieve their pending commands.
- **Security & Rate Limiting:** Implements basic rate limiting to prevent DDoS or spam telemetry, secured behind properly configured Security Groups.

#### PostgreSQL on AWS RDS (Database)

A managed relational database deployed within a private subnet, accepting inbound rules exclusively from the EC2 backend.

- **Schema Management:** Uses Alembic for schema migrations, managing relational tables for Buildings, Telemetry History, and Commands.
- **Command Queueing:** Implements database logic to securely queue pending commands for specific edge devices.
- **Performance Optimization:** Utilizes database indexing to optimize latency and API response times for historical data retrieval.

#### Python Simulator & AWS CloudWatch (IoT & Monitoring)

A combination of simulated edge computing and cloud-native monitoring to ensure the system operates reliably.

- **IoT Device Simulation:** Python scripts act as YOLO Uno or ESP32 edge devices, generating randomized, realistic sensor data using threading to simulate simultaneous multi-building traffic.
- **Device Polling & Two-Way Sync:** The simulator periodically polls (GET) pending commands from the backend and acknowledges execution, establishing full bi-directional communication.
- **Audit Trails & Monitoring:** AWS CloudWatch is integrated to monitor API error rates (HTTP 200/500 successes and errors) and log all executed commands for security audit trails.