---
title: "Week 8 Worklog"
date: "2026-07-27"
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Objectives:

- Connect all 5 layers of the architecture (Frontend, Backend, Database, Monitoring, Simulator).
- Ensure flawless, cohesive data flow from the Python Simulator to the React Dashboard and back.
- Prepare documentation for the eventual hardware transition (YOLO Uno).

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :-- | :--- | :--------- | :-------------- | :----------------- |
| 1   | **E2E Flow Test:** Run Dashboard, EC2 Backend, and Simulator simultaneously. Trace data packets to ensure smooth integration. | 03/08/2026 | 04/08/2026 | System Architecture Flow |
| 2   | **Latency Optimization:** Analyze API response times. Add DB indexing in PostgreSQL if necessary to improve query speeds. | 05/08/2026 | 05/08/2026 | PostgreSQL Indexing Guide |
| 3   | **Bug Squashing:** Fix edge cases (e.g., simulator disconnects, UI desync, malformed JSON handling). | 06/08/2026 | 07/08/2026 | QA Testing Guidelines |
| 4   | **YOLO Uno Prep:** Refine simulator documentation to clearly outline how the YOLO Uno hardware will replace it later. | 08/08/2026 | 09/08/2026 | Hardware Specs Docs |
| 5   | **Monitoring Review:** Verify CloudWatch logs capture all API successes (HTTP 200) and errors (HTTP 500) accurately. | 08/08/2026 | 09/08/2026 | AWS CloudWatch Metrics |

### Week 8 Achievements:

- **Full System Integration:** System functions cohesively as a single, fully integrated Enterprise IoT Cloud solution.
- **Optimized Performance:** Analyzed API latency and optimized PostgreSQL queries to ensure real-time responsiveness.
- **System Resilience:** Successfully squashed critical bugs related to UI desynchronization and simulator disconnects.
- **Transition Ready:** Completed transition documentation detailing the replacement of the Python simulator with physical YOLO Uno edge devices.