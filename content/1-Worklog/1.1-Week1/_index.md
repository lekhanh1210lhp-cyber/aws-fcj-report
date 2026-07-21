---
title: "Week 1 Worklog"
date: "2025-09-09"
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Week 1 Objectives:

- Analyze the business requirements for a **Multi-branch Building Management System (BMS)**.
- Design the **5-Layer System Architecture** for the Enterprise IoT Cloud Dashboard.
- Select and evaluate appropriate **AWS Cloud Services** (EC2, RDS, CloudWatch) for the backend infrastructure.
- Define the **Data Models** (Telemetry, Command, History) and establish the REST API specifications.
- Set up the project repository, version control, and assign roles for the engineering team.

### Tasks to be carried out this week:

| Day | Task                                                                                                                                                                                                                                                                                                                                                                                                                   | Start Date | Completion Date |
| :-- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------- | :-------------- |
| 1   | **Business Requirement Analysis** <br> - **Problem Definition:** Analyzed the need for centralized monitoring and control of smart buildings across multiple locations (Hanoi, Da Nang, HCMC). <br> - **System Scope:** Defined the required IoT sensors (temperature, humidity, light) and controllable devices (fans, lights, curtains).                                                                               | 08/09/2025 | 08/09/2025      |
| 2   | **System Architecture Design** <br> - **5-Layer Model:** Designed the end-to-end architecture covering IoT Devices (Simulator/YOLO Uno) ➔ HTTP REST API ➔ FastAPI (EC2) ➔ PostgreSQL ➔ React Dashboard. <br> - **Data Flow:** Mapped out the telemetry upstream (Device to Cloud) and command downstream (Cloud to Device).                                                                                            | 09/09/2025 | 09/09/2025      |
| 3   | **Cloud Infrastructure Planning** <br> - **Service Selection:** Evaluated AWS EC2 for hosting the FastAPI backend and AWS RDS (PostgreSQL) for time-series and relational data storage. <br> - **Monitoring Strategy:** Planned the integration of AWS CloudWatch for backend logging and system health monitoring.                                                                                                    | 10/09/2025 | 10/09/2025      |
| 4   | **API & Database Schema Definition** <br> - **Database Design:** Drafted the PostgreSQL schema for tracking device telemetry histories and command execution states. <br> - **API Contracts:** Documented the core REST API endpoints (e.g., `POST /telemetry`, `POST /command`) including request/response JSON payloads.                                                                                             | 11/09/2025 | 11/09/2025      |
| 5   | **Project Initialization & Git Setup** <br> - **Repository Setup:** Initialized the GitHub repository, established branching strategies, and created the foundational `README.md`. <br> - **Role Delegation:** Clarified responsibilities among team members (Cloud, Backend, Frontend, and IoT Engineers) to ensure smooth collaboration.                                                                             | 12/09/2025 | 12/09/2025      |

### Week 1 Achievements:

- **Finalized System Architecture**
  - Successfully laid out a robust, scalable 5-layer IoT architecture.
  - Clearly separated hardware concerns from backend logic, allowing future transition from Python Simulator to physical ESP32/YOLO Uno without backend modification.

- **Established Technical Specifications**
  - Completed initial Database schema design for PostgreSQL.
  - Defined strict JSON payloads for device-to-cloud communication.

- **Project Readiness**
  - Git repository is fully configured.
  - Team members have a clear understanding of their technical roles and upcoming development phases.

---

👉 **Outcome:** After Week 1, the theoretical foundation and architectural blueprint of the Enterprise IoT Dashboard are fully established. The team is now ready to begin coding the Python Simulator and Backend APIs in Week 2.