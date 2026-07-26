---

### 2. File tiếng Việt (`_index.vi.md`)

```markdown
---
title: "Tích hợp Ứng dụng Frontend"
date: "2026-06-15"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Mục tiêu

Bạn sẽ khởi tạo và cấu hình một **Giao diện Web Dashboard (GUI)** chuyên nghiệp để quản lý tòa nhà thông minh tập trung.

Chúng ta sử dụng:

- **Frontend Framework:** React (Vite).
- **Styling:** TailwindCSS.
- **API Client:** Axios.
- **Backend Target:** FastAPI trên AWS EC2.

#### Các Bước Thực hiện

**Phần I: Cấu hình Môi trường Node.js**

**Bước 1: Cài đặt Node.js & npm**

Mở Terminal trên máy tính của bạn để đảm bảo bạn có môi trường thực thi JavaScript cần thiết để chạy dự án Vite-React.

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