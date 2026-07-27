---
title: "Week 7 Worklog"
date: "2026-07-27"
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:

- Implement historical data visualization within the Frontend Dashboard.
- Develop and integrate remote device control interfaces (Control Panel) for administrators.
- Ensure administrators can view trends and dispatch commands directly from the UI to the backend.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :-- | :--- | :--------- | :-------------- | :----------------- |
| 1   | **Data Visualization Setup:** Review requirements for plotting historical temperature and humidity data. | 27/07/2026 | 28/07/2026 | Chart.js/Recharts Docs |
| 2   | **Control Panel Design:** Review requirements for building remote toggle switches in the UI. | 29/07/2026 | 29/07/2026 | React UI Components |
| 3   | **Data Visualization & Control Panel Implementation:** <br> - **Data Visualization:** Integrated Chart.js/Recharts to plot historical temperature and humidity data. <br> - **Control Panel:** Built toggle switches for Fan, Lights, and Curtains in the UI. | 30/07/2026 | 01/08/2026 | Frontend Framework Docs |
| 4   | **Command Dispatch & State Sync:** <br> - **Command Dispatch:** Bound UI toggles to Axios POST requests to the `/command` endpoint. <br> - **State Synchronization:** Implemented UI loading states to wait for IoT device acknowledgment. | 31/07/2026 | 01/08/2026 | Axios & React State Docs |
| 5   | **UX Refinement:** Polished dashboard aesthetics to ensure an "Enterprise BMS" look and feel. | 02/08/2026 | 02/08/2026 | UI/UX Design Guidelines |

### Week 7 Achievements:

- **Analytics Dashboard:** Successfully integrated charts (Chart.js/Recharts) to visualize historical telemetry trends.
- **Interactive Control Panel:** Built a fully functional control panel allowing admins to toggle Fan, Lights, and Curtains.
- **Command Integration:** Connected UI components to the backend `/command` API using Axios POST requests.
- **Enhanced UX:** Implemented synchronous loading states while waiting for device acknowledgment and refined the overall "Enterprise BMS" aesthetic.