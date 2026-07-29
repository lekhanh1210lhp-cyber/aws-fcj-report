---
title: "Tích hợp frontend"
date: "2026-07-28"
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Tổng quan và mục tiêu

Chạy dashboard React + Vite + TypeScript + Tailwind CSS trên máy cục bộ, chuyển các lời gọi API tới EC2, hiển thị telemetry, lịch sử và trạng thái máy chủ, đồng thời tạo lệnh có thể theo dõi mà không gửi trùng.

## Bước 1 - Kiểm tra và chạy dự án

Mã nguồn đã kiểm tra sử dụng React 19.2.7, Vite 8.1.1, TypeScript 6.0.2, Tailwind CSS 3.4.19, Axios, Recharts và Framer Motion. Từ Windows PowerShell:

```powershell
git clone <REPOSITORY_URL>
Set-Location .\aws-iot-dashboard\frontend
npm install
npm run dev
```

Dùng phiên bản Node theo yêu cầu của `package.json` và lockfile. Giữ nguyên lockfile, không thay file chỉ để xử lý khác biệt phiên bản trên máy. Mã nguồn lấy telemetry mới nhất và lịch sử sau mỗi 3 giây.

## Bước 2 - Cấu hình Vite proxy

Dùng đường dẫn tương đối `/api` trong các component. Proxy của môi trường phát triển giúp tránh lặp lại URL EC2:

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

Khởi động lại `npm run dev` sau khi thay đổi cấu hình Vite. Nếu dự án dùng `VITE_API_BASE_URL`, hãy định nghĩa biến trong `.env.local` đã được loại khỏi Git và đọc qua `import.meta.env`; không viết cứng URL trong nhiều thành phần.

File `vite.config.ts` đã kiểm tra hiện chứa một địa chỉ EC2 thật. Đây là vấn đề về bảo mật và khả năng bảo trì: cần thay bằng giá trị giữ chỗ hoặc cách cấu hình ở trên, đồng thời không sao chép địa chỉ đó vào báo cáo hay bằng chứng.

## Bước 3 - Kết nối với API đã tài liệu hóa

Với `deviceId = "room_01"`, UI dùng:

```text
GET  /api/devices/room_01/latest
GET  /api/devices/room_01/history
POST /api/devices/room_01/commands
```

Dùng `/openapi.json` để tạo hoặc xác minh kiểu dữ liệu TypeScript. Ánh xạ các trường từ máy chủ nhưng không đổi tên giá trị ánh sáng analog thành Lux. Thẻ dữ liệu mới nhất và biểu đồ lịch sử phải thể hiện rõ trạng thái đang tải, lỗi có thể thử lại và thời điểm cập nhật cuối.

Chỉ báo **Live AWS status** phải dựa trên yêu cầu kiểm tra sức khỏe hoặc API thực tế. Không hiển thị màu xanh chỉ vì ứng dụng React đã tải xong.

## Bước 4 - Xây dựng bảng điều khiển

Hiển thị các nút:

- `FAN_ON` / `FAN_OFF`;
- `LIGHT_ON` / `LIGHT_OFF`; và
- `CURTAIN_OPEN` / `CURTAIN_CLOSE`.

Mã nguồn hiện tại bắt lỗi API lệnh nhưng lại cập nhật trạng thái mô phỏng và trả về thành công; đồng thời chưa có cơ chế chặn yêu cầu đang gửi hoặc lệnh đang chờ. Cần sửa hành vi này trước khi dùng giao diện làm bằng chứng nghiệm thu:

1. vô hiệu hóa nút điều khiển đang chọn trong khi gửi POST;
2. chặn yêu cầu trùng khi một lệnh cùng loại vẫn đang chờ;
3. trả về thất bại khi POST lỗi thay vì cập nhật trạng thái mô phỏng của thiết bị;
4. hiển thị ID và trạng thái lệnh do máy chủ trả về;
5. làm mới lệnh/telemetry đến khi thấy ACK; và
6. hiển thị lỗi mà không tuyên bố thiết bị vật lý đã hoạt động.

Sau khi tải lại trình duyệt, trạng thái phải được khôi phục từ backend, không lấy từ nút chuyển chế độ cục bộ.

## Bước 5 - Ý nghĩa của chế độ và đề xuất

Nút chuyển chế độ gửi `MODE_AUTO` hoặc `MODE_MANUAL`. Chế độ tự động của firmware mới là nơi thực hiện điều khiển theo ngưỡng đã mô tả ở 5.6. Các đề xuất trên frontend chỉ là luật `if/else` định sẵn; vì vậy nhãn **AI Auto Control** không chính xác và nên đổi thành **Automatic rule-based control**.

Giao diện hiện lưu chế độ ở phía cục bộ, trong khi API chưa có endpoint trả về chế độ của firmware. Sau khi tải lại trang hoặc khi yêu cầu lỗi, giao diện và thiết bị có thể hiển thị khác nhau. Không được xem trạng thái của nút chuyển là trạng thái firmware đã xác nhận cho đến khi API cung cấp dữ liệu đó.

Khi lấy dữ liệu thất bại, `iotEngine.ts` chuyển sang dữ liệu sinh ngẫu nhiên có nhãn `SIMULATED`, còn giao diện dùng cụm “FAIL-PROOF.” Dữ liệu mô phỏng phải được phân biệt rõ, không được dùng làm bằng chứng vận hành; tuyên bố “không thể lỗi” nên được thay bằng nhãn thể hiện đúng chế độ suy giảm hoặc demo. Đồng thời, cần đổi nhãn **Lux** trong giao diện thành **Analog light value** cho đến khi có phép quy đổi đã hiệu chuẩn.

## Bước 6 - Xác minh lưu lượng trên trình duyệt

Mở DevTools → **Network**:

1. tải lại trang và xem các yêu cầu dữ liệu mới nhất/lịch sử;
2. tạo một lệnh;
3. kiểm tra phương thức, route số nhiều, nội dung yêu cầu, mã trạng thái và phản hồi JSON;
4. quan sát `Pending`, sau đó là trạng thái `Executed` do ACK; và
5. mô phỏng backend lỗi, xác nhận UI vẫn dùng được.

## Kết quả mong đợi

Telemetry và lịch sử được hiển thị, trạng thái AWS phản ánh yêu cầu backend thật, bảng điều khiển tạo đúng một lệnh có thể theo dõi và giao diện phân biệt rõ việc máy chủ nhận yêu cầu với việc thiết bị vật lý thực thi. Dữ liệu mô phỏng có nhãn rõ ràng; yêu cầu POST lỗi không được báo là đã điều khiển thành công.

<!-- TODO IMAGE: /images/5-Workshop/5.7-frontend/dashboard-overview.png — Dashboard hiển thị latest telemetry, history, nguồn dữ liệu real/simulated rõ ràng và Analog light value; che địa chỉ EC2. -->
<!-- TODO IMAGE: /images/5-Workshop/5.7-frontend/control-panel-api-request.png — Bảng điều khiển cùng thẻ Network của DevTools hiển thị một yêu cầu POST tới route số nhiều, ID lệnh và trạng thái máy chủ; che tên máy/địa chỉ IP. -->

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| Vite proxy 404 | Proxy key/target, plural path, restart Vite |
| Lỗi CORS | Yêu cầu có đi vòng qua proxy hay không và chính sách CORS của backend đã đầy đủ chưa |
| Biểu đồ trống | Cấu trúc phản hồi, timestamp và cách xử lý lịch sử rỗng |
| Trạng thái luôn báo online | Liên kết trạng thái với `/api/health`, không dựa vào lúc component được gắn |
| Lệnh bị lặp | Vô hiệu hóa nút đang gửi và kiểm tra lệnh/trạng thái đang chờ |
| Giao diện báo thành công quá sớm | Hiển thị `Pending` cho đến khi backend ghi nhận ACK/`Executed` |
| Mất kết nối sau khi EC2 khởi động lại | Cập nhật IP công khai mới hoặc dùng endpoint ổn định trong tương lai |

Tiếp theo: [chạy xác minh end-to-end](../5.8-End-to-End-Testing/).
