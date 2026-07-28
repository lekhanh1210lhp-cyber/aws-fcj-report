---
title: "Kết quả, thách thức và cải tiến tương lai"
date: "2026-07-28"
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

## Tổng quan

Mục này tách phần implementation đã xác minh bằng source khỏi evidence thực thi còn cần bổ sung. Source ứng dụng đã được review tại `F:\aws-iot-dashboard`; export deployment, ảnh CloudWatch, capture database và artifact test phần cứng không có trong report repository. Vì vậy các mục dưới đây vẫn là phát biểu nghiệm thu cần xác nhận theo mục 5.8, không phải kết quả test tự bịa.

## Kết quả cần xác nhận

| Kết quả | Bằng chứng nghiệm thu |
| :--- | :--- |
| Telemetry end-to-end | Request YOLO UNO, FastAPI response/log, RDS row, latest view trên dashboard |
| Dashboard và history | Latest card, history/chart theo thứ tự, loading/error behavior |
| PostgreSQL persistence | Query telemetry và command trước/sau API refresh |
| Điều khiển fan/light/curtain | Command ID ghép với evidence thiết bị vật lý |
| `Pending` → `Executed` | Create response và query sau cho cùng command ID |
| Command ACK | Serial line thiết bị, backend log và final state đã lưu |
| CloudWatch monitoring | Backend log mới, metric EC2/RDS và cấu hình alarm |

Chỉ đánh dấu hoàn tất khi đã đính kèm evidence. Prototype không chứng minh HA, latency dưới 50 ms, fail-proof, HTTPS, authentication hoặc AI control.

## Phát hiện khi review source

- Backend không có validator cho danh sách command hỗ trợ; giá trị không hợp lệ có thể được lưu ở `Pending`.
- Polling command trả bản ghi pending cũ nhất theo FIFO, dù route có tên `commands/latest`.
- ACK tìm theo command ID mà không xác minh device ID trên route hoặc state trước đó.
- Setting `DEVICE_API_KEY` có giá trị mặc định nhưng các route không enforce.
- Khi fetch lỗi, frontend có thể chuyển sang dữ liệu `SIMULATED`; command lỗi vẫn có thể được trình bày như mock state thành công.
- Mode của frontend là local state, chưa được API xác nhận; nhãn “AI” và “FAIL-PROOF” nói quá hành vi rule-based/demo.
- Frontend gọi ADC ánh sáng thô là Lux và hard-code EC2 target thật trong Vite config.
- Một phần mô tả hardware trong source nói servo GPIO 8 và bỏ LCD, nhưng firmware đang hoạt động dùng GPIO 38 và có LCD1602. Workshop lấy code đang hoạt động làm chuẩn.

## Thách thức và bài học

| Problem | Root cause | Solution | Lesson learned |
| :--- | :--- | :--- | :--- |
| SSH key bị từ chối trên Windows | Sai key path/ACL hoặc login user | Dùng đúng AMI user và giới hạn quyền key local | Chẩn đoán identity trước khi đổi network rule |
| Sai lệnh biến môi trường | PowerShell, CMD, Bash có cú pháp khác | Chỉ dùng `$env:...`, `%...%`, `$HOME` trong đúng shell | Ghi môi trường cho mọi command |
| Không tới được port 8000 | SG đóng hoặc Uvicorn bind `127.0.0.1` | Mở source đã duyệt, bind `0.0.0.0` cho demo | Test health local trước public path |
| RDS SSL lỗi | Sai CA path, hostname hoặc `DATABASE_URL` | Dùng bundle hiện hành và absolute path; kiểm tra endpoint | Network và TLS là hai lớp riêng |
| `systemd` thất bại | Sai user/path/module/environment | Xem status/journal và bám lệnh chạy tay thành công | Chỉ đưa lệnh đã xác minh vào unit |
| Vite proxy 404/CORS | Sai target/path hoặc bypass proxy | Dùng relative `/api`, restart Vite, xem Network | Giữ một cấu hình API base |
| Public IP thay đổi | EC2 stop/start nhận địa chỉ mới | Cập nhật cấu hình local/device | Endpoint ổn định là việc tương lai |
| Endpoint không đồng nhất | Route singular/plural hoặc client cũ | Lấy OpenAPI và source làm chuẩn | Chia sẻ một API contract có version |
| Command trùng | Poll/refresh/retry gửi hoặc chạy hai lần | Kiểm tra pending state và last command ID | ACK retry không được lặp actuation |
| ACK nhanh làm mất `Pending` | Polling nhanh thực thi ngay | Lưu create response và final state cùng ID | Evidence phải theo entity ID |
| Giá trị ánh sáng không chính xác | ADC thô chưa hiệu chuẩn | Gọi là analog value và hiệu chuẩn sau | Không tự đặt Lux |
| CloudWatch Agent không có data | Lỗi IAM, path, dimension hoặc config | Xem agent log và nguồn log thật | Permission và collection config khác nhau |

## Cải tiến tương lai

- Gắn Elastic IP hoặc dùng domain để có endpoint ổn định.
- Thêm Nginx hoặc reverse proxy đã review.
- Thêm HTTPS, authentication và authorization chặt hơn.
- Lưu application secret trong giải pháp managed secrets.
- Hỗ trợ nhiều thiết bị/phòng với rule ownership/authorization.
- Đánh giá WebSocket hoặc MQTT cho update ít overhead hơn.
- Đánh giá AWS IoT Core như lựa chọn messaging tương lai; hiện chưa triển khai.
- Containerize khi phù hợp và định nghĩa hạ tầng bằng code.
- Tự động hóa deployment và rollback đã test.
- Thêm alarm notification đã review.
- Hiệu chuẩn cảm biến ánh sáng và công bố phương pháp/unit quy đổi.

Mỗi future item cần owner, architecture review, cost/security analysis, implementation và test trước khi đưa vào sơ đồ current state.

## Kết quả

Sau khi review evidence, ghi Passed, Failed hoặc Not Run cho từng phát biểu nghiệm thu và link issue owner cho mọi gap. Không đổi “expected” thành “achieved” nếu chưa có bằng chứng.

Tiếp theo: [chuẩn bị bàn giao dự án](../5.12-Project-Handover/).
