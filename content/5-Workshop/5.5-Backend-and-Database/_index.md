---
title: "Backend Deployment and Database Integration"
date: "2026-07-28"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Overview and objectives

Install the Python application on Amazon Linux EC2, connect it to the private `iot_dashboard` database, verify the source-defined API, and keep Uvicorn running as `aws-iot-backend`. The application runbook uses user `ec2-user`, backend path `/home/ec2-user/aws-iot-dashboard/backend`, virtual environment `venv`, and entry point `main:app`.

## Step 1 - Connect and install prerequisites

From Windows PowerShell:

```powershell
ssh -i "$env:USERPROFILE\.ssh\<KEY_FILE>.pem" ec2-user@<EC2_PUBLIC_IP>
```

On Amazon Linux EC2, run in Linux Bash:

```bash
sudo dnf update -y
sudo dnf install -y git python3 python3-pip postgresql15 curl
```

If the selected Amazon Linux release exposes a different PostgreSQL client package name, confirm it with `dnf search postgresql` before installing.

## Step 2 - Clone and create the virtual environment

In EC2 Linux Bash:

```bash
git clone <REPOSITORY_URL> ~/aws-iot-dashboard
cd ~/aws-iot-dashboard/backend
python3 -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

The checked source has `backend/main.py` and exports `app`; therefore the Uvicorn entry point is `main:app`.

## Step 3 - Create the environment file

Create an ignored `.env` on EC2:

```dotenv
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@<RDS_ENDPOINT>:5432/iot_dashboard
```

If the backend uses `sslmode=verify-full`, download the current Amazon RDS CA bundle following the project runbook, store it with restricted permissions, and use the absolute path expected by SQLAlchemy/psycopg:

```dotenv
DATABASE_URL=postgresql://<DB_USER>:<DB_PASSWORD>@<RDS_ENDPOINT>:5432/iot_dashboard?sslmode=verify-full&sslrootcert=<ABSOLUTE_CA_PATH>/global-bundle.pem
```

URL-encode reserved characters in `<DB_PASSWORD>`. Never commit `.env` or the real password.

## Step 4 - Test PostgreSQL

From EC2 Linux Bash:

```bash
psql "host=<RDS_ENDPOINT> port=5432 dbname=iot_dashboard user=<DB_USER> sslmode=require"
```

In PostgreSQL `psql`:

```sql
SELECT current_database(), current_user;
\dt
```

Initialize the SQLAlchemy schema with the checked source command:

```bash
python -m app.database.init_db
```

It calls `Base.metadata.create_all` and should create `devices`, `telemetry_logs`, and `commands`. Confirm with `\dt` and `\d <table_name>`; this project does not define an Alembic migration workflow.

## Step 5 - Run Uvicorn manually

In EC2 Linux Bash, from the backend directory:

```bash
source venv/bin/activate
uvicorn main:app --host 0.0.0.0 --port 8000
```

In a second EC2 session:

```bash
curl -i http://127.0.0.1:8000/api/health
curl -s http://127.0.0.1:8000/openapi.json
```

Confirm the eight documented routes in section 5.3 and inspect the generated Pydantic request schemas before creating telemetry or command examples.

## Step 6 - Create `aws-iot-backend.service`

Create `/etc/systemd/system/aws-iot-backend.service` using the verified user, path, and Uvicorn module:

```ini
[Unit]
Description=AWS IoT FastAPI backend
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=ec2-user
WorkingDirectory=/home/ec2-user/aws-iot-dashboard/backend
EnvironmentFile=/home/ec2-user/aws-iot-dashboard/backend/.env
ExecStart=/home/ec2-user/aws-iot-dashboard/backend/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=on-failure
RestartSec=5
StandardOutput=append:/var/log/aws-iot-backend/backend.log
StandardError=append:/var/log/aws-iot-backend/backend-error.log

[Install]
WantedBy=multi-user.target
```

Prepare the log directory and start the service:

```bash
sudo install -d -o ec2-user -g ec2-user /var/log/aws-iot-backend
sudo systemctl daemon-reload
sudo systemctl enable aws-iot-backend
sudo systemctl restart aws-iot-backend
sudo systemctl status aws-iot-backend --no-pager
curl -i http://127.0.0.1:8000/api/health
```

**Expected result:** the service is `active (running)`, health returns HTTP 200, and application tables are present in `iot_dashboard`.

## Step 7 - Inspect logs and deploy an update

```bash
sudo journalctl -u aws-iot-backend -n 100 --no-pager
sudo tail -n 100 /var/log/aws-iot-backend/backend.log
cd ~/aws-iot-dashboard
git status --short
git pull --ff-only
source backend/venv/bin/activate
pip install -r backend/requirements.txt
sudo systemctl restart aws-iot-backend
curl -i http://127.0.0.1:8000/api/health
```

Pull only from the approved branch, rerun `python -m app.database.init_db` when models change, and review schema compatibility before restarting. Do not use `git reset --hard` as a deployment shortcut.

## Expected Result

The `aws-iot-backend` service is `active (running)`, `GET /api/health` returns HTTP 200, EC2 reaches private RDS, and `devices`, `telemetry_logs`, and `commands` exist in `iot_dashboard`. Deployment evidence records the application commit and contains no credentials.

<!-- TODO IMAGE: /images/5-Workshop/5.5-backend-database/backend-systemd-health-check.png — Terminal showing aws-iot-backend active and GET /api/health returning HTTP 200; redact hostnames, public IPs, and credentials. -->
<!-- TODO IMAGE: /images/5-Workshop/5.5-backend-database/postgresql-tables-and-commands.png — psql evidence for devices, telemetry_logs, and commands plus a redacted command-state query; do not expose the RDS endpoint or password. -->

## Troubleshooting

| Symptom | Diagnosis and correction |
| :--- | :--- |
| Connection refused | Confirm RDS/port or that Uvicorn is running and listening |
| Connection timeout | Check RDS SG source `iot-ec2-sg`, subnet routing, endpoint, and region |
| Wrong `DATABASE_URL` | Verify database/user/encoding; load the same `.env` used by systemd |
| Local curl works, remote fails | Bind `0.0.0.0`, verify EC2 SG port 8000 and public IP |
| `systemd` fails | Run `systemctl status` and `journalctl`; verify user, path, module, permissions |
| Port already in use | Use `sudo ss -ltnp | grep :8000` and stop the unintended process |
| SSL verification fails | Use the correct CA bundle, absolute path, permissions, and endpoint hostname |
| Tables missing | Run the source-defined migration/init process; do not create an ad-hoc schema |

Next: [integrate YOLO UNO hardware](../5.6-hardware-integration/).
