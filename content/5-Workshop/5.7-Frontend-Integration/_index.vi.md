---
title: "Tích hợp frontend"
date: "2026-07-28"
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Tổng quan và mục tiêu

Chạy dashboard React + Vite + TypeScript + Tailwind CSS tại local, định tuyến API call tới EC2, hiển thị telemetry/history và trạng thái server, đồng thời tạo command có thể theo dõi mà không gửi lặp.

## Bước 1 - Kiểm tra và chạy project

Source đã kiểm tra dùng React 19.2.7, Vite 8.1.1, TypeScript 6.0.2, Tailwind CSS 3.4.19, Axios, Recharts và Framer Motion. Từ Windows PowerShell:

```powershell
git clone <REPOSITORY_URL>
Set-Location .\aws-iot-dashboard\frontend
npm install
npm run dev
```

Dùng Node version mà `package.json`/lockfile yêu cầu. Giữ lockfile và không thay chỉ để xử lý khác biệt version local. Source polling latest telemetry và history mỗi 3 giây.

## Bước 2 - Cấu hình Vite proxy

Dùng relative path `/api` trong component. Development proxy giúp không lặp EC2 URL:

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

Restart `npm run dev` sau khi đổi cấu hình Vite. Nếu project dùng `VITE_API_BASE_URL`, định nghĩa trong `.env.local` đã ignore và đọc qua `import.meta.env`; không hard-code URL ở nhiều component.

File `vite.config.ts` đã kiểm tra hiện chứa một địa chỉ EC2 thật. Đây là vấn đề bảo mật và maintainability: thay bằng placeholder/cấu hình ở trên, không sao chép địa chỉ đó vào báo cáo hoặc evidence.

## Bước 3 - Kết nối API đã tài liệu hóa

Với `deviceId = "room_01"`, UI dùng:

```text
GET  /api/devices/room_01/latest
GET  /api/devices/room_01/history
POST /api/devices/room_01/commands
```

Dùng `/openapi.json` để sinh hoặc xác minh TypeScript type. Map field server mà không đổi tên giá trị ánh sáng analog thành Lux. Card latest và chart history cần loading state rõ ràng, error state có thể retry và thời điểm cập nhật cuối.

Chỉ báo **Live AWS status** phải dựa trên health/API request thật. Không hiện màu xanh chỉ vì ứng dụng React đã load.

## Bước 4 - Xây control panel

Hiển thị button cho:

- `FAN_ON` / `FAN_OFF`;
- `LIGHT_ON` / `LIGHT_OFF`; và
- `CURTAIN_OPEN` / `CURTAIN_CLOSE`.

Source hiện tại catch lỗi command API, cập nhật mock state và trả về thành công; source cũng chưa có guard cho request in-flight/command pending. Sửa hành vi này trước khi dùng UI làm bằng chứng nghiệm thu:

1. disable control đang chọn trong lúc POST;
2. chặn request giống nhau khi command cùng loại còn pending;
3. trả failure khi POST lỗi thay vì cập nhật mock actuator state;
4. hiện command ID và state do server trả;
5. refresh command/telemetry đến khi thấy ACK; và
6. hiển thị lỗi mà không tuyên bố actuator vật lý đã chạy.

Sau browser refresh, trạng thái phải được dựng lại từ backend, không lấy từ local toggle.

## Bước 5 - Ý nghĩa mode và recommendation

Toggle gửi `MODE_AUTO` hoặc `MODE_MANUAL`. Firmware auto mode mới thực hiện điều khiển threshold mô tả ở 5.6. Recommendation của frontend là rule `if/else` xác định trước; nhãn **AI Auto Control** vì vậy không chính xác và nên đổi thành **Automatic rule-based control**.

UI hiện lưu mode ở local trong khi API chưa có endpoint trả mode firmware. Sau refresh hoặc request lỗi, UI và thiết bị có thể lệch nhau. Không trình bày toggle state là trạng thái firmware đã xác nhận cho đến khi API contract cung cấp dữ liệu đó.

Khi fetch lỗi, `iotEngine.ts` chuyển sang dữ liệu sinh ngẫu nhiên có nhãn `SIMULATED`, còn giao diện dùng cụm “FAIL-PROOF.” Phải làm simulation dễ nhận biết, không dùng làm evidence vận hành và thay tuyên bố fail-proof bằng nhãn degraded/demo mode trung thực. Đồng thời đổi nhãn **Lux** trong source UI thành **Analog light value** đến khi có phép hiệu chuẩn.

## Bước 6 - Xác minh browser traffic

Mở DevTools → **Network**:

1. reload và xem request latest/history;
2. tạo một command;
3. kiểm tra method, route plural, request body, status và JSON response;
4. quan sát `Pending`, sau đó là trạng thái `Executed` do ACK; và
5. mô phỏng backend lỗi, xác nhận UI vẫn dùng được.

**Kết quả mong đợi:** telemetry và history được render, AWS health phản ánh backend, control tạo đúng một command có thể theo dõi, UI phân biệt nhận request với thực thi vật lý.

<!-- TODO IMAGE: /images/5-Workshop/5.7-frontend/dashboard-overview.png — Dashboard hiển thị latest telemetry, history, nguồn dữ liệu real/simulated rõ ràng và Analog light value; che địa chỉ EC2. -->
![Giao diện Trạm Điều khiển Thời gian thực hiển thị dữ liệu telemetry và Bảng Gửi Lệnh điều khiển thiết bị từ xa](/images/5-Workshop/5.7-frontend/dashboard-overview_1.PNG)
![Giao diện Phân tích & Đề xuất Tự động kèm Biểu đồ Lịch sử được truy vấn trực tiếp từ Amazon RDS PostgreSQL](/images/5-Workshop/5.7-frontend/dashboard-overview_2.PNG)
*Hình 5-7-1. Dashboard hiển thị latest telemetry, history, nguồn dữ liệu real/simulated rõ ràng và Analog light value.*
<!-- TODO IMAGE: /images/5-Workshop/5.7-frontend/control-panel-api-request.png — Control panel cùng DevTools Network hiển thị một POST route plural, command ID và server state; che host/IP. -->
![Kiểm tra giao tiếp API trên Web Dashboard qua Chrome DevTools Network tab, trả về HTTP 200 OK thành công.](/images/5-Workshop/5.7-frontend/control-panel-api-request.PNG)
*Hình 5-7-2. Kiểm tra giao tiếp API trên Web Dashboard qua Chrome DevTools Network tab, trả về HTTP 200 OK thành công.*

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| Vite proxy 404 | Proxy key/target, plural path, restart Vite |
| CORS error | Request bypass proxy hoặc backend CORS chưa đủ |
| Chart trống | Response shape, timestamp, xử lý history rỗng |
| Status luôn online | Bind với `/api/health`, không phải component mount |
| Command lặp | Disable control đang gửi và kiểm tra pending command/state |
| UI báo thành công quá sớm | Hiện `Pending` đến khi backend báo ACK/`Executed` |
| Chạy đến khi EC2 restart | Cập nhật public IP mới hoặc endpoint ổn định trong tương lai |

Tiếp theo: [chạy xác minh end-to-end](../5.8-End-to-End-Testing/).
