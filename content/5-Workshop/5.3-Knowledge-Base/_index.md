---
title: "Database Design & Backend Foundation"
date: "2026-06-15"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Objectives

After completing the cloud infrastructure foundation, the next step is to set up the core backend and database components of the IoT architecture. In this section, we will provision PostgreSQL on AWS RDS and initialize the FastAPI backend structure.

We will accomplish 3 key technical objectives:

1.  **Initialize Database:** Deploy a PostgreSQL RDS instance in a private subnet and configure inbound rules to exclusively accept traffic from the EC2 instance.
2.  **FastAPI Initialization:** Initialize the FastAPI project and configure SQLAlchemy alongside Pydantic schemas.
3.  **Database Migration:** Set up Alembic for schema migrations and execute the first migration to RDS to create relational tables for Buildings, Telemetry History, and Commands.

#### Key Components

During this configuration process, we will interact with and connect the following services and tools:

- **AWS RDS (PostgreSQL):** The managed relational database service used to securely store structured data, including Buildings, Telemetry History, and Commands.
- **FastAPI (Python):** The modern backend framework actively running on the EC2 instance, responsible for handling API logic and validating data.
- **Alembic & SQLAlchemy:** The database toolkit and migration tools used to set up relational schemas and execute deployments directly to the RDS instance.

#### Implementation Steps

1. [AWS RDS Setup](5.3.1-RDS-Setup/)
2. [FastAPI Init & Database Migration](5.3.2-FastAPI-Migration/)