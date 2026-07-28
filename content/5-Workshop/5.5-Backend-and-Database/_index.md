---
title: "Backend Deployment and Database Integration"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Backend Deployment and Database Integration

## Step 1 - Install the application

Connect to EC2, clone the backend repository, and create an isolated environment:

```bash
git clone <BACKEND_REPOSITORY_URL> ~/aws-iot-dashboard
cd ~/aws-iot-dashboard/backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Create `.env` locally on EC2 and keep it out of Git:

```dotenv
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@<RDS_ENDPOINT>:5432/iot_dashboard
```

Run the project’s migration or table-initialization command, then verify the tables with `psql`.

## Step 2 - Test Uvicorn

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000
curl http://127.0.0.1:8000/api/health
```

Open `/openapi.json` or `/docs` only from an approved source.

## Step 3 - Run with systemd

Create `/etc/systemd/system/aws-iot-backend.service` with the correct user, working directory, environment file, and `.venv` Uvicorn path. Send standard output and errors to the project’s configured log files, then run:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now aws-iot-backend
sudo systemctl status aws-iot-backend
```

**Expected result:** `aws-iot-backend` is active, `/api/health` returns HTTP 200, and application tables exist in RDS.

## Troubleshooting

- Use `journalctl -u aws-iot-backend -n 100` when manual Uvicorn works but `systemd` fails.
- URL-encode special characters in the database password and verify that the service can read `.env`.

Next: [connect YOLO UNO and the actuators](../5.6-Hardware-Integration/).
