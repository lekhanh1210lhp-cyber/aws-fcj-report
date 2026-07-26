---
title: "Frontend Dashboard Integration"
date: "2026-06-15"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Target

You will initialize and configure a professional **Web Dashboard GUI (Graphical User Interface)** for centralized smart building management. 

We use:

- **Frontend Framework:** React (Vite).
- **Styling:** TailwindCSS.
- **API Client:** Axios.
- **Backend Target:** FastAPI on AWS EC2.

#### Implementation Steps

**Part I: Configure Node.js Environment**

**Step 1: Install Node.js & npm**

Open Terminal on your computer to ensure you have the JavaScript runtime required to run the Vite-React project.

```bash
# macOS (using Homebrew)
brew install node

# Linux (Ubuntu/Debian)
sudo apt update
sudo apt install nodejs npm

node -v
npm -v

git clone [https://github.com/your-org/iot-dashboard-frontend.git](https://github.com/your-org/iot-dashboard-frontend.git)
cd iot-dashboard-frontend

npm install

VITE_API_BASE_URL="http://<YOUR_EC2_ELASTIC_IP>:8000"

npm run dev