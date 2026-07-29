---
title: "Kiểm thử và xác minh end-to-end"
date: "2026-07-28"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Tổng quan và mục tiêu

Xác minh từng ranh giới một cách độc lập, sau đó kiểm tra toàn bộ luồng telemetry và lệnh. Mã nguồn backend và firmware đã được rà soát để xác định schema cùng hành vi chính xác; trước khi kiểm thử, dùng FastAPI `/docs` hoặc `/openapi.json` để xác nhận đúng phiên bản đang triển khai.

## Bước 1 - Thiết lập quy trình kiểm thử

1. Ghi ngày, người kiểm thử, mã commit ứng dụng, phiên bản firmware, khu vực AWS và thiết bị `room_01`.
2. Che thông tin xác thực và endpoint riêng trong bằng chứng.
3. Thu thập yêu cầu/phản hồi, log liên quan, trạng thái SQL, đầu ra thiết bị và trạng thái dashboard.
4. Điền giá trị quan sát vào **Thực tế/bằng chứng** và chỉ đánh dấu **Đạt/Không đạt** sau khi thực hiện.
5. Đưa phần cứng và dịch vụ về trạng thái an toàn sau các phép kiểm thử lỗi.

## Bước 2 - Thực thi và ghi ma trận kiểm thử

| ID | Mục tiêu | Điều kiện trước | Các bước | Kết quả mong đợi | Thực tế/bằng chứng | Đạt/Không đạt |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| T01 | Sức khỏe backend | Dịch vụ đang hoạt động | `GET /api/health` | HTTP 200 và nội dung kiểm tra sức khỏe đúng định nghĩa | Lưu phản hồi curl | Ghi |
| T02 | POST telemetry | Biết schema OpenAPI; DB truy cập được | Gửi một payload `room_01` hợp lệ | Phản hồi thành công và lưu một bản ghi | Đính kèm API + SQL | Ghi |
| T03 | Telemetry mới nhất | Hoàn tất T02 | `GET /api/devices/room_01/latest` | Trả bản ghi mới nhất | Đính kèm phản hồi | Ghi |
| T04 | Lịch sử | Có nhiều bản ghi | `GET /api/devices/room_01/history` | Trả lịch sử đúng thiết bị và đúng thứ tự | Đính kèm phản hồi/biểu đồ | Ghi |
| T05 | Tạo lệnh | Không có thao tác trùng đang chờ | POST một lệnh được hỗ trợ | ID lệnh có trạng thái `Pending` | Đính kèm phản hồi | Ghi |
| T06 | Phần cứng thăm dò lệnh | Thiết bị trực tuyến | Quan sát quá trình thăm dò sau T05 | Thiết bị nhận đúng ID/lệnh đúng một lần | Bằng chứng Serial Monitor | Ghi |
| T07 | Bật/tắt quạt | Quạt được đấu nối an toàn | Gửi `FAN_ON`, rồi `FAN_OFF` | Trạng thái vật lý đúng với từng lệnh | Video/ảnh + ID | Ghi |
| T08 | Bật/tắt đèn | Đèn/relay được đấu nối an toàn | Gửi `LIGHT_ON`, rồi `LIGHT_OFF` | Trạng thái vật lý đúng với từng lệnh | Video/ảnh + ID | Ghi |
| T09 | Mở/đóng rèm | Servo được đấu nối an toàn | Gửi `CURTAIN_OPEN`, rồi `CURTAIN_CLOSE` | Servo đi đến vị trí được định nghĩa trong mã nguồn | Video/ảnh + ID | Ghi |
| T10 | Vòng đời ACK | Có lệnh từ T05-T09 | Quan sát POST ACK và truy vấn trạng thái | Cùng một lệnh đổi `Pending` → `Executed` | API + SQL + log | Ghi |
| T11 | Khả năng lưu bền vững PostgreSQL | Có phiên kết nối DB | Truy vấn sau khi gửi telemetry/lệnh | Bản ghi vẫn còn sau khi tải lại hoặc khởi động lại API | Bằng chứng SQL | Ghi |
| T12 | Log CloudWatch | Agent đã cấu hình | Tạo yêu cầu sức khỏe/telemetry mới | Sự kiện backend mới xuất hiện đúng luồng log | Bằng chứng CloudWatch | Ghi |
| T13 | Backend không hoạt động | Khoảng bảo trì an toàn | Dừng dịch vụ; thử lại từ máy khách; khởi động lại | Báo lỗi/thử lại rõ ràng, không báo thành công giả | Bằng chứng UI/thiết bị/log | Ghi |
| T14 | Mất kết nối Wi-Fi | Thiết bị ở trạng thái an toàn | Ngắt Wi-Fi, quan sát, kết nối lại | Kết nối lại thành công; không lặp lệnh | Bằng chứng Serial Monitor | Ghi |
| T15 | Lệnh không được hỗ trợ | Kiểm thử có kiểm soát; thiết bị chấp hành an toàn | Gửi giá trị không được hỗ trợ | Backend hiện có thể lưu thành `Pending`; firmware phải từ chối và không ACK. Ghi đây là lỗi kiểm tra đầu vào của backend, không phải phép kiểm thử 4xx đạt | API + SQL + log nối tiếp | Ghi |

## Bước 3 - Kiểm tra API và cơ sở dữ liệu

Từ EC2 Linux Bash:

```bash
curl -i http://127.0.0.1:8000/api/health
curl -s http://127.0.0.1:8000/api/devices/room_01/latest
curl -s http://127.0.0.1:8000/api/devices/room_01/history
```

Tạo telemetry bằng các trường camelCase ở 5.6. Tạo lệnh bằng `{ "command": "FAN_ON" }`; trường Pydantic cũng có alias `Command`, nhưng `populate_by_name=True` nên tên viết thường `command` vẫn được chấp nhận. Bản ghi thiết bị phải tồn tại trước, thường được tạo bởi yêu cầu telemetry đầu tiên. Trong PostgreSQL `psql`, kiểm tra trạng thái lệnh:

```sql
SELECT
    id,
    device_id,
    command,
    state
FROM commands
ORDER BY id DESC
LIMIT 6;
```

Thiết bị có thể thăm dò và ACK nhanh đến mức truy vấn sau không còn thấy `Pending`. Hãy lưu phản hồi POST thể hiện `Pending`, sau đó chụp bản ghi `Executed` có cùng ID.

## Bước 4 - Xác minh xử lý lỗi và nghiệm thu

Trong T13/T14, giao diện và firmware phải báo không khả dụng, không được tuyên bố thành công. Frontend hiện trả về thành công mô phỏng cho một số lệnh lỗi, nên T13 dự kiến sẽ phát hiện lỗi này cho đến khi hành vi được sửa. Việc gửi lại ACK không được làm thiết bị chấp hành hoạt động lần nữa. Backend không có bộ kiểm tra enum cho lệnh; vì vậy T15 phải được đánh dấu **Không đạt** nếu backend nhận giá trị sai. Việc firmware từ chối không làm cho phần kiểm tra đầu vào của backend trở thành đạt.

<!-- TODO IMAGE: /images/5-Workshop/5.8-testing/telemetry-api-database-validation.png — Yêu cầu/phản hồi telemetry khớp với truy vấn bảng telemetry_logs của room_01; che endpoint và thông tin xác thực. -->
<!-- TODO IMAGE: /images/5-Workshop/5.8-testing/command-pending-to-executed.png — Cùng một ID lệnh xuất hiện ở trạng thái Pending rồi chuyển sang Executed sau ACK trong bằng chứng API/SQL. -->
<!-- TODO IMAGE: /images/5-Workshop/5.8-testing/dashboard-hardware-control.png — Ghép bảng điều khiển với bằng chứng vật lý của quạt, đèn và rèm; hiển thị ID lệnh khi có thể và che thông tin mạng. -->

## Kết quả mong đợi

Mọi dòng T01-T15 phải có giá trị quan sát trong cột **Thực tế/bằng chứng** và trạng thái **Đạt**, **Không đạt** hoặc **Chưa chạy**. Một phép kiểm thử đầu cuối chỉ được xem là đạt khi liên kết được cùng ID thiết bị/lệnh qua API, PostgreSQL, firmware, dashboard và log liên quan. Các lỗi frontend/backend đã biết vẫn phải ghi là không đạt cho đến khi được sửa và chạy lại.

## Xử lý sự cố

Đây là kế hoạch thực hiện, không phải tuyên bố rằng các phép kiểm thử đã chạy. Không gọi đây là kiểm thử tải và không tự tạo số liệu về độ trễ, thông lượng hoặc độ tin cậy. Mỗi phép kiểm thử không đạt phải ghi rõ lớp xảy ra lỗi, log/yêu cầu làm bằng chứng, người phụ trách, cách khắc phục và kết quả chạy lại.

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| Không thấy `Pending` | Lưu phản hồi của yêu cầu POST tạo lệnh, sau đó truy vấn cùng ID sau ACK |
| Giao diện và cơ sở dữ liệu không khớp | Xác nhận nguồn dữ liệu thật/mô phỏng, route API số nhiều và bản ghi cơ sở dữ liệu mới nhất |
| Lệnh bị lặp | So sánh ID lệnh, `lastAck`, `pendingAck` và tách việc gửi lại ACK khỏi việc điều khiển thiết bị |
| Không tái tạo được phép kiểm thử | Ghi mã commit, khu vực, ID thiết bị, timestamp/múi giờ và điều kiện ban đầu chính xác |
| Bằng chứng có dữ liệu nhạy cảm | Che thông tin và chụp lại; thay mới bí mật đã lộ trước khi tiếp tục |

Tiếp theo: [cấu hình và xác minh CloudWatch](../5.9-CloudWatch-Monitoring/).
