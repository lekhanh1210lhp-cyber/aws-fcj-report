---
title: "Backend Deployment and Database Integration"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Overview
The objective of this section is to deploy the FastAPI backend onto the EC2 instance, establish a secure connection with the RDS PostgreSQL database, and ensure the service runs continuously in the background using `systemd`[cite: 2].

## Step 1 - Install the application

Connect to EC2, clone the backend repository, and create an isolated environment:

```bash
git clone <BACKEND_REPOSITORY_URL> ~/aws-iot-dashboard
cd ~/aws-iot-dashboard/backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

*Note: Ensure that you have already created the database (e.g., `iot_dashboard`) in your RDS instance during the AWS Infrastructure Setup (Section 5.4). Your PostgreSQL URL must contain this exact database name at the end.*

Create `.env` locally on EC2 and keep it out of Git:

```dotenv
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@<RDS_ENDPOINT>:5432/iot_dashboard
```

Test the direct connection to the RDS database using `psql` to verify network connectivity and security groups:

```bash
psql -h <RDS_ENDPOINT> -U <DB_USER> -d iot_dashboard
```

Run the database initialization script to migrate data and create the necessary tables:

```bash
python -m app.database.init_db
```

**Expected result:** Dependencies are installed and the application tables are successfully created in RDS.
![PostgreSQL tables](images/postgresql-tables.png)

## Step 2 - Test Uvicorn

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
curl "http://127.0.0.1:8000/api/health"
```

Open `/docs` to view the documentation of api.

**Expected result:** The `/api/health` endpoint returns an HTTP 200 status code, and the Swagger UI is accessible.
![curl health](images/curl-health.png)
![Swagger](images/swagger-ui.png)

## Step 3 - Run with systemd

Create `/etc/systemd/system/aws-iot-backend.service` with the correct user, working directory, environment file, and `.venv` Uvicorn path. Send standard output and errors to the project’s configured log files, then run:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now aws-iot-backend
sudo systemctl status aws-iot-backend
```

**Expected result:** `aws-iot-backend` is active, `/api/health` returns HTTP 200, and application tables exist in RDS.
![systemd active](images/systemd-active.png)
![backend logs](images/backend-logs.png)

## Troubleshooting

- Use `journalctl -u aws-iot-backend -n 100` when manual Uvicorn works but `systemd` fails.
- URL-encode special characters in the database password and verify that the service can read `.env`.

## Result
The FastAPI backend is fully deployed on EC2, securely connected to RDS PostgreSQL, and reliably managed by `systemd`. 

Next: [connect YOLO UNO and the actuators](../5.6-Hardware-Integration/).