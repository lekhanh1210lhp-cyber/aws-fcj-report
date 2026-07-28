---
title: "Database Design & Backend Foundation"
date: "2026-06-15"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Objectives

After preparing the AWS cloud environment, the next step is to establish the data layer and backend foundation of the IoT dashboard. This section focuses on building the database and API backbone that will later receive telemetry data from simulated devices and serve it to the frontend.

We will complete three main technical objectives:

1. **Provision the database layer** using Amazon RDS for PostgreSQL in a private subnet.
2. **Initialize the FastAPI backend** and define the core request/response models.
3. **Apply database migrations** so the system can store buildings, telemetry records, and commands in a structured way.

#### What we will build

In this section, the project will rely on the following services and tools:

- **AWS RDS (PostgreSQL):** A managed relational database used to store structured IoT data such as building information, telemetry history, and command logs.
- **FastAPI (Python):** A modern backend framework that handles API requests, validates payloads, and connects to the database.
- **SQLAlchemy and Alembic:** Tools for defining database models and applying schema changes safely.

#### Implementation workflow

1. Create the PostgreSQL database instance in AWS RDS and make sure it is reachable only from the EC2 backend.
2. Configure the FastAPI service so it can connect to the database using environment variables and secure credentials.
3. Create the initial database schema and apply the first migration.
4. Verify that the tables are available and ready for telemetry ingestion.

<!-- Insert screenshot: AWS RDS instance created in a private subnet -->
> Placeholder for screenshot: AWS RDS PostgreSQL instance and security configuration.

#### Expected result

By the end of this section, the system should have a stable database and backend foundation ready to support API testing and frontend integration.