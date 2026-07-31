---
title: "Frontend Integration"
date: "2026-07-28"
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Overview and objectives

Run the React + Vite + TypeScript + Tailwind CSS dashboard locally, route API calls to EC2, display telemetry/history and server state, and create traceable commands without duplicate submissions.

## Step 1 - Inspect and start the project

The checked source uses React 19.2.7, Vite 8.1.1, TypeScript 6.0.2, Tailwind CSS 3.4.19, Axios, Recharts, and Framer Motion. From Windows PowerShell:

```powershell
git clone <REPOSITORY_URL>
Set-Location .\aws-iot-dashboard\frontend
npm install
npm run dev
```

Use the Node version required by `package.json`/lockfile. Keep the lockfile and do not replace it simply to resolve a local version mismatch. The source polls latest telemetry and history every 3 seconds.

## Step 2 - Configure the Vite proxy

Use relative `/api` paths in components. A development proxy avoids duplicating the EC2 URL:

```ts
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      "/api": {
        target: "http://<EC2_PUBLIC_IP>:8000",
        changeOrigin: true,
      },
    },
  },
});
```

Restart `npm run dev` after changing Vite configuration. If the project instead uses `VITE_API_BASE_URL`, define it in an ignored `.env.local` and read it through `import.meta.env`; do not hard-code the URL across components.

The checked `vite.config.ts` currently contains a real EC2 address. Treat that as a security and maintainability issue: replace it with the placeholder/configuration pattern above and never copy the address into this report or evidence.

## Step 3 - Bind the documented API

For `deviceId = "room_01"`, the UI uses:

```text
GET  /api/devices/room_01/latest
GET  /api/devices/room_01/history
POST /api/devices/room_01/commands
```

Use `/openapi.json` to generate or verify TypeScript types. Map server fields without renaming an analog light value to Lux. Latest cards and history charts should show an explicit loading state, a retryable error state, and a last-updated timestamp.

The **Live AWS status** indicator must be based on a real health/API request. It must not show green solely because the React application loaded.

## Step 4 - Build the control panel

Expose buttons for:

- `FAN_ON` / `FAN_OFF`;
- `LIGHT_ON` / `LIGHT_OFF`; and
- `CURTAIN_OPEN` / `CURTAIN_CLOSE`.

The current source catches command API failures, mutates mock state, and returns success. It also lacks an in-flight/pending-command guard. Correct that behavior before using the UI as acceptance evidence:

1. disable the selected control while the POST is in flight;
2. block an identical request while a matching command is still pending;
3. return failure when the POST fails instead of mutating mock actuator state;
4. show the command ID and server-returned state;
5. refresh command/telemetry state until ACK is visible; and
6. display failure without optimistically claiming physical execution.

Browser refresh must reconstruct state from the backend, not from a local toggle.

## Step 5 - Mode and recommendation semantics

The toggle sends `MODE_AUTO` or `MODE_MANUAL`. Firmware auto mode performs the actual threshold control described in 5.6. The frontend's recommendations are deterministic `if/else` rules; the label **AI Auto Control** is therefore inaccurate and should be changed to **Automatic rule-based control**.

The current UI stores mode locally, while the API has no endpoint that reports firmware mode. After refresh or a failed request, UI and device can diverge. Do not present the toggle state as confirmed firmware state until the API contract exposes it.

On data-fetch errors, `iotEngine.ts` falls back to generated data marked `SIMULATED`, and the interface uses the phrase “FAIL-PROOF.” Keep simulation visually unmistakable, never treat it as operational evidence, and replace the fail-proof claim with an honest degraded/demo-mode label. Also change the source UI label **Lux** to **Analog light value** until a calibrated conversion exists.

## Step 6 - Verify browser traffic

Open browser DevTools → **Network**:

1. reload and inspect latest/history requests;
2. create one command;
3. verify method, plural route, request body, status, and JSON response;
4. observe `Pending`, then the ACK-driven `Executed` state; and
5. simulate a backend error and confirm the UI remains usable.

**Expected result:** telemetry and history render, AWS health reflects the backend, controls create one traceable command, and the UI distinguishes request acceptance from physical execution.

<!-- TODO IMAGE: /images/5-Workshop/5.7-frontend/dashboard-overview.png — Dashboard showing latest telemetry, history, explicit real/simulated data source, and Analog light value; redact the EC2 address. -->
![Dashboard showing Real-time Control Station interface displaying telemetry data and the Remote Control Panel](/images/5-Workshop/5.7-frontend/dashboard-overview_1.PNG)
![Automated Analysis & Recommendation Panel with Historical Telemetry Charts queried directly from Amazon RDS PostgreSQL](/images/5-Workshop/5.7-frontend/dashboard-overview_2.PNG)
*Figure 5-7-1. Dashboard showing latest telemetry, history, explicit real/simulated data source, and Analog light value.*
<!-- TODO IMAGE: /images/5-Workshop/5.7-frontend/control-panel-api-request.png — Control panel together with DevTools Network showing one plural-route POST, command ID, and server state; redact host/IP details. -->
![API communication verification on the Web Dashboard via Chrome DevTools Network tab, with HTTP 200 OK responses](/images/5-Workshop/5.7-frontend/control-panel-api-request.PNG)
*Figure 5-7-2. API communication verification on the Web Dashboard via Chrome DevTools Network tab, with HTTP 200 OK responses.*
## Troubleshooting

| Symptom | Check |
| :--- | :--- |
| Vite proxy 404 | Proxy key/target, plural path, Vite restart |
| CORS error | Request bypasses the proxy or backend CORS policy is incomplete |
| Blank chart | Response shape, timestamps, empty-history handling |
| Status is always online | Bind it to `/api/health`, not component mount |
| Repeated commands | Disable in-flight controls and check pending command/state |
| UI says success too early | Display `Pending` until backend reports ACK/`Executed` |
| Works until EC2 restarts | Update the changed public IP or future stable endpoint |

Next: [run end-to-end validation](../5.8-End-to-End-Testing/).
