---
title: "FastAPI Init & Database Migration"
date: "2026-06-15"
weight: 2
chapter: false
pre: " <b> 5.3.2 </b> "
---

#### Target

Before the backend can receive telemetry data, we must initialize the FastAPI project and prepare the database structure. We will configure SQLAlchemy and Pydantic schemas, set up Alembic, and execute the first migration to the PostgreSQL RDS instance.

#### Implementation Steps

**Step 1: Initialize FastAPI & Schemas**

The Backend Engineer will first set up the core backend structure.

1.  Initialize the FastAPI project environment in your local workspace or EC2 instance.
    ![FastAPI Init](/images/5-Workshop/5.3-FastAPI-Migration/13_FastAPI_Init.jpg)

2.  Configure Pydantic schemas. This ensures the backend can validate incoming JSON data payloads and reject malformed telemetry data later on.
    ![Pydantic Schemas](/images/5-Workshop/5.3-FastAPI-Migration/14_Pydantic.jpg)

3.  Set up SQLAlchemy models to define the relational tables. You will need to design tables for Buildings, Telemetry History, and Commands.
    ![SQLAlchemy Models](/images/5-Workshop/5.3-FastAPI-Migration/15_SQLAlchemy.jpg)

**Step 2: Configure Alembic**

Now we will prepare the database migration environment to deploy our schemas to AWS RDS.

1.  Set up Alembic for schema migrations.
2.  Configure the Alembic `alembic.ini` and `env.py` files to point the database connection string to your newly deployed PostgreSQL RDS instance.
3.  Generate the initial migration script based on the SQLAlchemy models you defined.

![Alembic Setup](/images/5-Workshop/5.3-FastAPI-Migration/16_Alembic_Setup.jpg)

**Step 3: Execute Migration to RDS**

Finally, we apply the database schema to the live RDS instance to complete the backend foundation.

1.  Execute the first migration command (e.g., `alembic upgrade head`) to push the schema changes to RDS.
2.  Connect to your PostgreSQL database (using tools like pgAdmin or DBeaver) to verify the deployment.
3.  **Result:**
    - The database schema is successfully deployed.
    - Relational tables (Buildings, Telemetry History, Commands) are properly created and actively running.

![Migration Success](/images/5-Workshop/5.3-FastAPI-Migration/17_Migration_Success.jpg)

**Congratulations!** You have successfully initialized the FastAPI backend structure and deployed the database schema. The backend server is now prepared for the next phase: REST API Implementation for data ingestion.