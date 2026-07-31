---
title: "Kiểm thử và Xác thực End-to-End"
date: "2026-07-28"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Tổng quan và mục tiêu

Xác thực từng thành phần của hệ thống một cách độc lập trước khi thực hiện quy trình truyền dữ liệu (telemetry) và điều khiển thiết bị (command) theo luồng end-to-end hoàn chỉnh. Mã nguồn backend và firmware đã được kiểm tra là cơ sở xác định cấu trúc API và hành vi của các lệnh điều khiển. Trước khi kiểm thử, cần xác nhận phiên bản FastAPI đang được triển khai thông qua `/docs` hoặc `/openapi.json`.

---

## Bước 1 - Thiết lập quy trình kiểm thử

1. Ghi lại ngày kiểm thử, người thực hiện, commit ID của ứng dụng, phiên bản firmware, vùng AWS và mã thiết bị (`room_01`).
2. Che hoặc loại bỏ thông tin nhạy cảm như thông tin xác thực, API key và các endpoint nội bộ khỏi toàn bộ bằng chứng kiểm thử.
3. Thu thập đầy đủ request/response của API, log của backend, trạng thái cơ sở dữ liệu SQL, đầu ra của thiết bị, trạng thái dashboard và log CloudWatch (nếu có).
4. Ghi nhận kết quả quan sát được vào cột **Actual/Evidence** và chỉ đánh dấu **Pass**, **Fail** hoặc **Not Run** sau khi quá trình kiểm thử hoàn tất.
5. Sau các bài kiểm thử lỗi, khôi phục phần cứng và các dịch vụ về trạng thái hoạt động an toàn.

---

## Bước 2 - Thực hiện và ghi nhận ma trận kiểm thử

| ID | Mục tiêu | Điều kiện tiên quyết | Các bước thực hiện | Kết quả mong đợi | Bằng chứng thực tế | Pass/Fail |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| T01 | Kiểm tra trạng thái backend | Dịch vụ đang hoạt động | `GET /api/health` | HTTP 200 và nội dung phản hồi đúng theo tài liệu | Phản hồi HTTP 200 và log backend | **Pass** |
| T02 | Gửi telemetry | Đã biết schema OpenAPI; cơ sở dữ liệu sẵn sàng | Gửi một payload telemetry hợp lệ của `room_01` | API trả về thành công và lưu đúng một bản ghi | Hình 15: bản ghi API và SQL khớp nhau | **Pass** |
| T03 | Lấy telemetry mới nhất | Đã hoàn thành T02 | `GET /api/devices/room_01/latest` | Trả về bản ghi telemetry mới nhất | Đã xác minh phản hồi API | **Pass** |
| T04 | Lấy lịch sử telemetry | Có nhiều bản ghi trong cơ sở dữ liệu | `GET /api/devices/room_01/history` | Trả về lịch sử telemetry theo đúng thứ tự thời gian | Phản hồi API và biểu đồ trên dashboard | **Pass** |
| T05 | Tạo lệnh điều khiển | Không có lệnh Pending trùng lặp | Gửi một lệnh điều khiển hợp lệ | Sinh Command ID với trạng thái `Pending` | Hình 16: Command ID 189 ở trạng thái `Pending` | **Pass** |
| T06 | Kiểm tra cơ chế polling của phần cứng | Thiết bị đang trực tuyến | Quan sát sau khi thực hiện T05 | Thiết bị nhận đúng lệnh và chỉ nhận một lần | Video minh họa phần cứng | **Pass** |
| T07 | Điều khiển Quạt BẬT/TẮT | Quạt được kết nối an toàn | Gửi `FAN_ON`, sau đó `FAN_OFF` | Trạng thái thực tế của quạt khớp với lệnh | Dashboard và bằng chứng phần cứng | **Pass** |
| T08 | Điều khiển Đèn BẬT/TẮT | Đèn được kết nối an toàn | Gửi `LIGHT_ON`, sau đó `LIGHT_OFF` | Trạng thái thực tế của đèn khớp với lệnh | Dashboard và bằng chứng phần cứng | **Pass** |
| T09 | Điều khiển Rèm MỞ/ĐÓNG | Servo được kết nối an toàn | Gửi `CURTAIN_OPEN`, sau đó `CURTAIN_CLOSE` | Servo quay đến 90° rồi trở về 0° | Dashboard và bằng chứng phần cứng | **Pass** |
| T10 | Kiểm tra vòng đời ACK | Đã có lệnh từ T05-T09 | Quan sát ACK và kiểm tra trạng thái | Cùng một Command ID chuyển từ `Pending` sang `Executed` | Hình 16: Command ID 189 chuyển sang `Executed` | **Pass** |
| T11 | Kiểm tra tính lưu trữ của PostgreSQL | Cơ sở dữ liệu sẵn sàng | Truy vấn sau khi gửi telemetry và command | Dữ liệu vẫn tồn tại sau khi API refresh hoặc restart | Bằng chứng SQL | **Pass** |
| T12 | Kiểm tra CloudWatch | CloudWatch Agent đã được cấu hình | Tạo một request API mới | Log mới xuất hiện trong CloudWatch | Log CloudWatch (Mục 5.9) | **Pass** |
| T13 | Backend không khả dụng | Trong thời gian bảo trì an toàn | Dừng backend, gửi lại request, khởi động lại | Client hiển thị lỗi, không báo thành công giả | Giao diện, API và log | **Record** |
| T14 | Mất kết nối Wi-Fi | Thiết bị ở trạng thái an toàn | Ngắt Wi-Fi rồi kết nối lại | Thiết bị kết nối lại và không thực hiện lệnh trùng lặp | Bằng chứng từ Serial Monitor | **Pass** |
| T15 | Lệnh không được hỗ trợ | Môi trường kiểm thử được kiểm soát | Gửi một lệnh không hợp lệ | Backend hiện vẫn chấp nhận yêu cầu nhưng firmware từ chối thực thi và không gửi ACK. Ghi nhận đây là lỗi xác thực của backend. | API + SQL + Serial Monitor | **Fail (dự kiến)** |

---

## Bước 3 - Kiểm tra API và cơ sở dữ liệu

Từ EC2 Linux Bash:

```bash
curl -i http://127.0.0.1:8000/api/health
curl -s http://127.0.0.1:8000/api/devices/room_01/latest
curl -s http://127.0.0.1:8000/api/devices/room_01/history
```

Tạo telemetry bằng các trường dữ liệu theo chuẩn camelCase được định nghĩa trong Mục 5.6. Tạo một lệnh điều khiển như sau:

```json
{
  "command": "FAN_ON"
}
```

Mô hình Pydantic cũng hỗ trợ tên trường Command; tuy nhiên, do populate_by_name=True, trường command viết thường vẫn được chấp nhận. Một bản ghi thiết bị phải tồn tại trước đó, thông thường được tạo bởi request telemetry đầu tiên.

Kiểm tra trạng thái các lệnh trong PostgreSQL:

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

Do thiết bị có thể gửi ACK gần như ngay lập tức sau khi nhận lệnh, trạng thái Pending có thể biến mất rất nhanh. Vì vậy, cần lưu lại phản hồi POST thể hiện trạng thái Pending trước khi chụp trạng thái cuối cùng là Executed của cùng Command ID.

---

### Bằng chứng T02 - Telemetry được lưu vào Amazon RDS

Hình 15 minh họa một request curl được thực hiện trong môi trường kiểm soát nhằm tách riêng việc kiểm tra khả năng lưu dữ liệu từ FastAPI xuống PostgreSQL. Việc tích hợp với phần cứng đã được kiểm thử riêng trước đó; hình này chỉ dùng để xác thực trường hợp kiểm thử T02.

![Telemetry submitted through the API and stored in PostgreSQL](/images/5-Workshop/5.8-testing/telemetry-api-database-validation.png)

*Hình 15. Dữ liệu telemetry được gửi thông qua REST API và được lưu thành công vào Amazon RDS for PostgreSQL.*

---

### Bằng chứng T05/T10 - Vòng đời của lệnh điều khiển

Để tách riêng việc kiểm thử backend trong thời điểm phần cứng chưa sẵn sàng, lệnh FAN_ON được tạo thông qua REST API và endpoint ACK được gọi thủ công. Kết quả cho thấy cùng một Command ID (189) chuyển từ trạng thái Pending sang Executed, qua đó xác thực luồng xử lý giữa backend và cơ sở dữ liệu mà không phụ thuộc vào phần cứng.

![Command 189 changing from Pending to Executed after ACK](/images/5-Workshop/5.8-testing/command-pending-to-executed.png)

*Hình 16. Xác thực vòng đời của lệnh điều khiển từ Pending sang Executed thông qua endpoint ACK của FastAPI.*

---

### Bằng chứng T06-T09 - Dashboard và phần cứng

Các ảnh chụp dashboard dưới đây minh họa việc gửi thành công các lệnh điều khiển từ xa cho quạt, đèn và rèm cửa. Phản hồi thực tế của các thiết bị được ghi lại trong video minh họa.

![Remote fan control through dashboard](/images/5-Workshop/5.8-testing/dashboard-hardware-control_Fan.PNG)

![Remote light control through dashboard](/images/5-Workshop/5.8-testing/dashboard-hardware-control_LED.PNG)

![Remote curtain control through dashboard](/images/5-Workshop/5.8-testing/dashboard-hardware-control_Curtain.PNG)

*Hình 17. Giao diện dashboard dùng để gửi các lệnh điều khiển từ xa cho quạt, hệ thống chiếu sáng và rèm cửa.*

Video minh họa đầy đủ được lưu tại Google Drive:

https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing

Video này cung cấp bằng chứng end-to-end cho thấy các lệnh từ dashboard được ESP32 nhận thành công và điều khiển chính xác các thiết bị vật lý gồm quạt, đèn và rèm cửa.

---

## Bước 4 - Xử lý lỗi và tiêu chí chấp nhận

Các trường hợp kiểm thử lỗi (T13-T15) nhằm xác minh rằng hệ thống xử lý và thông báo lỗi đúng cách, đồng thời không tạo ra trạng thái thành công giả.

- Trong trường hợp backend ngừng hoạt động (T13), giao diện người dùng và firmware phải hiển thị trạng thái không khả dụng thay vì báo lệnh thực hiện thành công.
- Khi mất kết nối Wi-Fi (T14), firmware phải tự động kết nối lại và tiếp tục polling mà không thực hiện lại các lệnh trước đó.
- Đối với các lệnh không được hỗ trợ (T15), backend lý tưởng nên từ chối ngay từ bước xác thực. Tuy nhiên, do backend hiện chưa kiểm tra tập lệnh hợp lệ, việc chấp nhận các giá trị không hợp lệ cần được ghi nhận là một lỗi xác thực của backend.

---

## Kết quả mong đợi

Mỗi trường hợp kiểm thử từ T01 đến T15 cần bao gồm:

- Kết quả thực tế quan sát được.
- Trạng thái Pass, Fail hoặc Not Run.
- Bằng chứng hỗ trợ như phản hồi API, truy vấn SQL, đầu ra của phần cứng, ảnh chụp dashboard hoặc log CloudWatch.

Một quy trình kiểm thử end-to-end được xem là thành công khi cùng một Device ID và Command ID có thể được đối chiếu nhất quán trên REST API, PostgreSQL, firmware, dashboard và hệ thống giám sát.

---

## Khắc phục sự cố

Phần này mô tả hướng dẫn thực hiện kiểm thử, không phải khẳng định rằng tất cả các bài kiểm thử đã được chạy. Không mô tả đây là kiểm thử hiệu năng (stress/performance testing) và không đưa ra các số liệu về độ trễ hoặc thông lượng nếu chúng chưa được đo đạc riêng.

| Triệu chứng | Cách kiểm tra |
| :--- | :--- |
| Không quan sát được trạng thái Pending | Lưu phản hồi POST trước khi ACK và truy vấn lại cùng Command ID sau đó |
| Dashboard và cơ sở dữ liệu không đồng nhất | Kiểm tra endpoint API, nguồn dữ liệu của dashboard và bản ghi mới nhất trong cơ sở dữ liệu |
| Lệnh được thực hiện nhiều lần | So sánh Command ID, lastAck và pendingAck; phân biệt việc gửi lại ACK với việc thực thi lại lệnh |
| Backend báo thành công dù đang gặp lỗi | Kiểm tra xử lý lỗi của frontend và trạng thái hoạt động của backend |
| Backend chấp nhận lệnh không hợp lệ | Ghi nhận đây là lỗi xác thực của backend cho đến khi chức năng kiểm tra lệnh được triển khai |
| Không thể tái hiện bài kiểm thử | Ghi lại commit ID, vùng AWS, Device ID, thời gian và toàn bộ điều kiện tiên quyết |
| Bằng chứng chứa thông tin nhạy cảm | Che thông tin nhạy cảm và chụp lại trước khi công bố |

Tiếp theo: [Cấu hình và xác thực CloudWatch](../5.9-CloudWatch-Monitoring/).