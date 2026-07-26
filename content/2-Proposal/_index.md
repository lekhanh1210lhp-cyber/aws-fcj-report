---
title: "Proposal"
date: "2026-06-15"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Enterprise IoT Cloud Dashboard

### You can read the full proposal here: <a href="/files/2-Proposal/IoT_Dashboard_Proposal.pdf" download>IoT Dashboard Proposal</a>

### 1. Executive Summary

The project is an Enterprise IoT Cloud Dashboard on AWS designed for centralized smart building management (BMS). The system utilizes React for the frontend, FastAPI on AWS EC2 for the backend, PostgreSQL RDS for the database, CloudWatch for monitoring, and a Python Simulator for IoT devices. The project development started on June 15, 2026. 

### 2. Problem Statement

#### Current Problem & Solution
The project addresses the need for centralized smart building management (BMS) through a cloud-based approach. The solution involves finalizing a 5-layer architecture. It incorporates telemetry data ingestion—receiving temperature, humidity, light, and device status—and enables remote command execution (e.g., Fan ON/OFF, Curtain OPEN).

#### Benefits
The system establishes full bi-directional communication between the Simulator and the Cloud. It ensures that edge devices, simulating YOLO Uno or ESP32 hardware, can reliably generate and send telemetry to the backend, while also fetching pending commands.

### 3. Solution Architecture

The architecture consists of 5 layers, managed by assigned team roles. 

#### Tech Stack Details:
| Component | Technology | Details |
| :--- | :--- | :--- |
| **Frontend** | **React** | Uses Vite-React, TailwindCSS, and routing. Integrates Axios for API calls and Chart.js/Recharts for data visualization. |
| **Backend** | **FastAPI (Python)** | Hosted on Ubuntu EC2. Uses SQLAlchemy and Pydantic schemas. |
| **Database** | **PostgreSQL** | Deployed on AWS RDS with Alembic for schema migrations. |
| **IoT / Edge** | **Python Simulator** | Simulates YOLO Uno/ESP32 edge devices using the 'requests' and 'threading' libraries. |
| **Monitoring** | **CloudWatch** | Integrated to monitor API error rates and audit trails for executed commands. |

#### AWS Workflow:
1. **Networking:** Involves VPC design with public/private subnets, an Internet Gateway, and Route Tables.
2. **Compute:** Ubuntu EC2 instance is launched for the FastAPI backend, attached to an Elastic IP, with HTTP/SSH Security Groups.
3. **Database Security:** PostgreSQL RDS is deployed in a private subnet with inbound rules strictly configured from EC2 only.
4. **Identity:** AWS IAM Setup includes users, groups, policies, and enforces MFA for all accounts.

### 4. Technical Deployment

#### Implementation Phases
The project spans exactly 10 weeks:
1. **Foundation (Weeks 1-2):** Set up AWS environment, VPC, EC2, and PostgreSQL RDS. Initialize FastAPI and database schemas.
2. **Data & IoT (Weeks 3-5):** Implement telemetry APIs, Python simulator, and establish bi-directional communication for command execution.
3. **Frontend UI (Weeks 6-7):** Build React dashboard, integrate API, and visualize historical data.
4. **Integration & Security (Weeks 8-10):** Execute End-to-End flow tests, optimize latency, harden infrastructure, and finalize LaTeX documentation.

#### Detailed Technical Requirements:
- **CI/CD Draft:** Utilize deployment scripts to pull code and restart systemctl services on EC2.
- **Data Validation:** Implement Pydantic validators to reject malformed data payloads.
- **Command Queue:** PostgreSQL logic queues pending commands for specific devices, which the simulator polls periodically.

### 5. Timeline & Milestones (Sprints)

- **Week 1:** System Architecture & Cloud Infrastructure Foundation.
- **Week 2:** Database Design & Backend Foundation.
- **Week 3:** REST API Implementation (Data Ingestion).
- **Week 4:** IoT Device Simulation.
- **Week 5:** Command Execution & Two-Way Communication.
- **Week 6:** Frontend Development - Dashboard UI.
- **Week 7:** Frontend Development Control Panel & Analytics.
- **Week 8:** End-to-End System Integration.
- **Week 9:** System Security & Stress Testing.
- **Week 10:** Final Documentation & Pitch Preparation.

### 6. Budget Estimation

The project emphasizes cost optimization. 
- **Resource Sizing:** Ensuring resources are properly sized using t2.micro or t3.micro instances.
- **Monitoring:** Reviewing AWS billing alerts to maintain financial control.

### 7. Risk Assessment

- **Security Risks:** Addressed via Security Audits to ensure Security Groups isolate the DB from the public internet.
- **DDoS/Spam:** Mitigated by implementing basic API Rate Limiting in FastAPI.
- **System Load:** Evaluated through Stress Testing, where the simulator sends high-frequency data while monitoring EC2 CPU.
- **Network Drops:** Handled by adding retry logic and exception handling within the IoT Simulator for dropped connections.

### 8. Expected Results & Team

#### Expected Results of the Project
- **Cohesive Operation:** The system functions cohesively as a single Enterprise IoT Cloud solution.
- **Hardware Readiness:** Refined simulator documentation outlines how the YOLO Uno will eventually replace the software simulation.
- **Final Output:** The project is ready for academic submission and a professional 30-second enterprise pitch presentation.

#### Implementation Team Roles:
The project architecture and tasks are strictly assigned to specific roles:
| Role | Primary Responsibilities |
| :--- | :--- |
| **Cloud Engineer** | Sets up IAM, VPC, EC2, RDS, and CloudWatch monitoring. Reviews Security Groups. |
| **Backend Engineer** | Initializes FastAPI, database schemas, Alembic migrations, and CORS resolution. |
| **IoT Engineer** | Creates Python simulator scripts, multithreading, and high-frequency stress testing. |
| **Frontend Engineer** | Creates Vite-React project, TailwindCSS layout, API integration, and dashboard aesthetics. |