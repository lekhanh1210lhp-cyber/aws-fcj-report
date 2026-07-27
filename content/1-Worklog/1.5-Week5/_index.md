---
title: "Week 5 Worklog"
date: "2026-07-13"
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:

- Enable the backend to receive command requests from the Dashboard.
- Develop mechanisms for IoT edge devices to fetch and execute pending commands.
- Establish full bi-directional communication between the Python Simulator and the Cloud architecture.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :-- | :--- | :--------- | :-------------- | :----------------- |
| 1   | **POST /command:** Create API endpoint for Dashboard to send commands (e.g., Fan ON/OFF, Curtain OPEN). | 13/07/2026 | 14/07/2026 | FastAPI Docs |
| 2   | **Command Queue:** Implement logic in PostgreSQL to queue pending commands for specific devices. | 15/07/2026 | 15/07/2026 | PostgreSQL Docs |
| 3   | **Device Polling:** Update Python Simulator to GET pending commands periodically and acknowledge execution. | 16/07/2026 | 17/07/2026 | Python Requests Docs |
| 4   | **Command Logging:** Ensure all executed commands are logged in CloudWatch for audit trails. | 18/07/2026 | 19/07/2026 | AWS CloudWatch Docs |
| 5   | **System Review:** Team sync to ensure backend is robust before frontend integration. | 18/07/2026 | 19/07/2026 | System Architecture Docs |

### Week 5 Achievements:

- **Bi-directional Communication:** Full bi-directional communication established between Simulator and Cloud.
- **Command Endpoints:** Successfully created API endpoints for remote device control operations (Fan, Curtain).
- **Queue Mechanism:** Designed a reliable PostgreSQL queuing logic for managing pending IoT device commands.
- **Audit Logging:** Integrated CloudWatch to maintain complete audit trails for all executed commands.