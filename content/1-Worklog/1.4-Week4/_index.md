---
title: "Week 4 Worklog"
date: "2026-07-06"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

- Develop the **Python Simulator** to act as the YOLO Uno/ESP32 edge devices for the Enterprise IoT Cloud Dashboard.
- Establish a reliable mechanism to generate and transmit simulated telemetry data directly to the EC2 backend.
- Ensure system robustness through multi-threading, error handling, and end-to-end integration testing.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :-- | :--- | :--------- | :-------------- | :----------------- |
| 1   | **Simulator Design** <br> - **Data Generation:** IoT Engineer created Python scripts to generate randomized, realistic sensor data. | 06/07/2026 | 07/07/2026 | Python `random` Docs |
| 2   | **HTTP Client Setup** <br> - **API Communication:** Implemented the Python `requests` library to POST JSON data to the FastAPI public IP on EC2. | 08/07/2026 | 08/07/2026 | Python `requests` Docs |
| 3   | **Multi-Building Simulation** <br> - **Scaling:** Scaled the simulator script using threading to simulate traffic from HN, DN, and HCM buildings simultaneously. | 09/07/2026 | 10/07/2026 | Python `threading` Docs |
| 4   | **Error Handling** <br> - **Network Resilience:** Added retry logic and exception handling to gracefully manage dropped network connections during transmission. | 11/07/2026 | 11/07/2026 | System Architecture Docs |
| 5   | **Integration Test** <br> - **End-to-End Verification:** Verified that telemetry data from all simulated buildings successfully and accurately appears in the PostgreSQL database. | 12/07/2026 | 12/07/2026 | PostgreSQL Docs |

### Week 4 Achievements:

- **IoT Simulation Success:** Python scripts are reliably generating and sending telemetry data to the EC2 backend.
- **Scalability Achieved:** Successfully simulated concurrent data streams from multiple building locations (HN, DN, HCM) using threading.
- **System Reliability:** Improved the simulator's stability by implementing robust error handling and retry logic for network drops.
- **Verified Data Flow:** Confirmed full integration where simulated sensor data is properly stored in the backend PostgreSQL database.