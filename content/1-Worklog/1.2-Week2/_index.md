---
title: "Week 2 Worklog"
date: "2026-06-22"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:

- Provision PostgreSQL on AWS RDS.
- Initialize the FastAPI backend structure.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date |
| :-- | :--- | :--------- | :-------------- |
| 1 | **Database Schema:** Design relational tables for Buildings, Telemetry History, and Commands. | 22/06/2026 | 22/06/2026 |
| 2 | **AWS RDS Setup:** Deploy PostgreSQL RDS instance in private subnet, configure inbound rules from EC2 only. | 23/06/2026 | 24/06/2026 |
| 3 | **FastAPI Init:** Backend Engineer initializes FastAPI project, configures SQLAlchemy and Pydantic schemas. | 25/06/2026 | 26/06/2026 |
| 4 | **Database Migration:** Set up Alembic for schema migrations and execute first migration to RDS. | 25/06/2026 | 27/06/2026 |
| 5 | **CI/CD Draft:** Draft deployment scripts to pull code and restart systemctl services on EC2. | 28/06/2026 | 28/06/2026 |

### Week 2 Achievements:

- Database schema deployed.
- Backend server actively running on EC2.

---

👉 **Outcome:** After Week 2, the relational database and foundational REST API structure are successfully established, preparing the system for data ingestion in Week 3.