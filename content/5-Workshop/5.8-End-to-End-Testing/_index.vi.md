---
title: "Kiểm thử và xác minh end-to-end"
date: "2026-07-28"
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Tổng quan và mục tiêu

Xác minh từng ranh giới độc lập, sau đó chạy toàn bộ luồng telemetry và command. Source backend và firmware đã kiểm tra xác định schema cùng hành vi chính xác; dùng FastAPI `/docs` hoặc `/openapi.json` để xác nhận bản đang deploy trước khi test.

## Quy trình kiểm thử

1. Ghi ngày, người test, commit ID ứng dụng, firmware build, AWS region và `room_01`.
2. Che credential và private endpoint trong evidence.
3. Thu request/response, log liên quan, trạng thái SQL, output thiết bị và trạng thái dashboard.
4. Điền giá trị quan sát vào **Actual/evidence** và chỉ đánh dấu **Pass/Fail** sau khi chạy.
5. Đưa phần cứng và service về trạng thái an toàn sau failure test.

## Ma trận kiểm thử

| ID | Mục tiêu | Điều kiện trước | Các bước | Kết quả mong đợi | Actual/evidence | Pass/Fail |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| T01 | Backend health | Service active | `GET /api/health` | HTTP 200 và health body đã định nghĩa | Ghi curl response | Ghi |
| T02 | POST telemetry | Biết OpenAPI schema; DB reachable | Post một payload `room_01` hợp lệ | Response thành công và một row được lưu | Đính kèm API + SQL | Ghi |
| T03 | Latest telemetry | Hoàn tất T02 | `GET /api/devices/room_01/latest` | Trả bản ghi mới nhất | Đính kèm response | Ghi |
| T04 | History | Có nhiều bản ghi | `GET /api/devices/room_01/history` | History theo thiết bị và thứ tự | Đính kèm response/chart | Ghi |
| T05 | Tạo command | Không có action pending trùng | POST một command hỗ trợ | Command ID có state `Pending` | Đính kèm response | Ghi |
| T06 | Hardware polling | Thiết bị online | Quan sát polling sau T05 | Thiết bị nhận đúng ID/command một lần | Serial evidence | Ghi |
| T07 | Fan ON/OFF | Quạt nối an toàn | Gửi `FAN_ON`, rồi `FAN_OFF` | Trạng thái vật lý đúng từng command | Video/ảnh + ID | Ghi |
| T08 | Light ON/OFF | Đèn/relay nối an toàn | Gửi `LIGHT_ON`, rồi `LIGHT_OFF` | Trạng thái vật lý đúng từng command | Video/ảnh + ID | Ghi |
| T09 | Curtain OPEN/CLOSE | Servo nối an toàn | Gửi open, rồi close | Servo tới vị trí source định nghĩa | Video/ảnh + ID | Ghi |
| T10 | Vòng đời ACK | Có command T05-T09 | Quan sát POST ACK và query state | Cùng command đổi `Pending` → `Executed` | API + SQL + log | Ghi |
| T11 | PostgreSQL persistence | Có DB session | Query sau telemetry/command | Bản ghi còn sau API refresh/restart | SQL evidence | Ghi |
| T12 | CloudWatch logs | Agent đã cấu hình | Tạo health/telemetry request mới | Event backend mới xuất hiện đúng stream | CloudWatch evidence | Ghi |
| T13 | Backend unavailable | Maintenance window an toàn | Stop service; retry client; restart | Báo lỗi/retry rõ, không báo thành công giả | UI/device/log evidence | Ghi |
| T14 | Wi-Fi disconnected | Trạng thái thiết bị an toàn | Ngắt Wi-Fi, quan sát, kết nối lại | Reconnect; không lặp command | Serial evidence | Ghi |
| T15 | Unsupported command | Test có kiểm soát; actuator an toàn | Gửi giá trị không hỗ trợ | Backend hiện tại có thể lưu thành `Pending`; firmware phải từ chối và không ACK. Ghi đây là lỗi validation backend, không phải test 4xx pass | API + SQL + serial log | Ghi |

## Kiểm tra API và database

Từ EC2 Linux Bash:

```bash
curl -i http://127.0.0.1:8000/api/health
curl -s http://127.0.0.1:8000/api/devices/room_01/latest
curl -s http://127.0.0.1:8000/api/devices/room_01/history
```

Tạo telemetry bằng các field camelCase ở 5.6. Tạo command bằng `{ "command": "FAN_ON" }`; Pydantic field cũng có alias `Command`, nhưng `populate_by_name=True` nên lowercase `command` được chấp nhận. Device row phải tồn tại trước, thông thường do telemetry request đầu tiên tạo. Trong PostgreSQL `psql`, kiểm tra command state:

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

Thiết bị polling có thể ACK nhanh đến mức query sau không còn thấy `Pending`. Hãy giữ POST response thể hiện `Pending`, rồi chụp bản ghi `Executed` cuối cùng có cùng ID.

## Xử lý lỗi và nghiệm thu

Trong T13/T14, UI và firmware phải báo unavailable mà không tuyên bố thành công. Frontend đã kiểm tra hiện trả simulated success cho một số command lỗi, nên T13 dự kiến sẽ lộ defect cho đến khi hành vi này được sửa. ACK retry không được lặp actuator action. Backend đã kiểm tra không có command enum validator, vì vậy đánh dấu T15 **Fail** khi backend nhận giá trị; firmware từ chối không làm backend validation thành pass.

<!-- TODO IMAGE: /images/5-Workshop/5.8-testing/telemetry-api-database-validation.png — Telemetry request/response khớp với query telemetry_logs của room_01; che endpoint và credential. -->
<!-- TODO IMAGE: /images/5-Workshop/5.8-testing/command-pending-to-executed.png — Cùng command ID xuất hiện trước ở Pending, sau ở Executed sau ACK trong evidence API/SQL. -->
<!-- TODO IMAGE: /images/5-Workshop/5.8-testing/dashboard-hardware-control.png — Control dashboard ghép với evidence vật lý cho quạt, đèn, rèm; hiện command ID khi có thể và che thông tin mạng. -->

## Kết quả và xử lý sự cố

Đây là kế hoạch thực thi, không phải tuyên bố test đã chạy. Không gọi là stress testing và không bịa số liệu latency, throughput hoặc reliability. Test fail phải ghi layer lỗi, log/request evidence, owner, cách sửa và kết quả rerun.

Tiếp theo: [cấu hình và xác minh CloudWatch](../5.9-CloudWatch-Monitoring/).
