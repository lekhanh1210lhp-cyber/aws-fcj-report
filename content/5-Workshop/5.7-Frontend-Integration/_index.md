---
title: "Frontend Integration"
date: "2026-07-28"
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# Frontend Integration

## Step 1 - Configure Vite

Install and start the frontend:

```bash
cd frontend
npm install
npm run dev
```

Configure the Vite development proxy so `/api` targets `http://<EC2_PUBLIC_IP>:8000`. Components should call relative paths such as `/api/telemetry/latest` instead of embedding the EC2 address throughout the code.

## Step 2 - Bind data and controls

Implement:

- latest readings and historical charts for `room_01`;
- manual controls for fan, light, and curtain;
- command status based on backend data, not an optimistic local toggle; and
- manual and automatic/recommendation modes.

If automatic suggestions are threshold rules, label them **rule-based recommendations**, not machine learning.

## Step 3 - Verify browser traffic

Use DevTools **Network** to confirm request URL, HTTP status, JSON response, and refresh behavior. A control click should create a command and display its server-provided status.

**Expected result:** Telemetry is visible, history updates, and each control creates a traceable `Pending` command.

## Troubleshooting

- A Vite 404 usually means an incorrect proxy path; CORS errors indicate that requests are bypassing the proxy or backend policy is incomplete.
- If the UI shows success before hardware execution, bind the display to command status and ACK instead of local component state.

Next: [validate the complete system](../5.8-End-to-End-Testing/).
