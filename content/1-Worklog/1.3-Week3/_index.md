---
title: "Week 3 Worklog"
date: "2026-06-29"
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:

- Build and test the telemetry ingestion API endpoints for the Enterprise IoT Cloud Dashboard.
- Design and implement RESTful API endpoints for telemetry data ingestion using FastAPI.
- Establish robust data validation mechanisms using Pydantic to reject malformed data payloads.
- Enable historical data retrieval with pagination.
- Ensure API reliability through comprehensive Postman testing and CloudWatch monitoring.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :-- | :--- | :--------- | :-------------- | :----------------- |
| 1   | **POST /telemetry:** Develop endpoint to receive temperature, humidity, light, and device status. | 29/06/2026 | 30/06/2026 | Backend API Docs |
| 2   | **Data Validation:** Implement Pydantic validators to reject malformed data payloads. | 01/07/2026 | 01/07/2026 | FastAPI & Pydantic Docs |
| 3   | **GET /telemetry:** Develop endpoints to retrieve latest status and historical data with pagination. | 02/07/2026 | 03/07/2026 | Backend API Docs |
| 4   | **Postman Testing:** Write and execute comprehensive Postman test suites for the API. | 04/07/2026 | 04/07/2026 | Postman Test Suite |
| 5   | **CloudWatch Logs:** Cloud Engineer integrates AWS CloudWatch to monitor API error rates. | 05/07/2026 | 05/07/2026 | AWS CloudWatch Docs |

### Week 3 Achievements:

- **API Development:** Backend successfully receives and validates JSON data.
- **Database Integration:** Validated JSON data is accurately stored in the PostgreSQL database.
- **Data Validation:** Implemented strict Pydantic schemas, successfully filtering out malformed IoT payloads.
- **Testing & Monitoring:** Completed API test suites in Postman and integrated AWS CloudWatch for real-time error rate tracking.