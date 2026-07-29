---
title: "Kết quả, thách thức và hướng cải tiến"
date: "2026-07-28"
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

## Tổng quan

Phần này tách rõ những nội dung đã được đối chiếu với mã nguồn và những nội dung vẫn cần bằng chứng từ môi trường chạy thực tế. Mã nguồn ứng dụng tại `F:\aws-iot-dashboard` đã được rà soát, nhưng kho báo cáo vẫn chưa có đầy đủ dữ liệu xuất từ môi trường triển khai, ảnh CloudWatch, ảnh cơ sở dữ liệu và kết quả kiểm thử phần cứng. Vì vậy, các mục dưới đây là điều kiện nghiệm thu cần xác nhận theo mục 5.8, không phải kết quả đã đạt.

## Kết quả cần xác nhận

| Kết quả | Bằng chứng nghiệm thu |
| :--- | :--- |
| Telemetry đầu cuối | Yêu cầu từ YOLO UNO, phản hồi/log FastAPI, bản ghi RDS và màn hình dữ liệu mới nhất trên dashboard |
| Dashboard và lịch sử | Thẻ dữ liệu mới nhất, lịch sử/biểu đồ đúng thứ tự, trạng thái tải và lỗi |
| Khả năng lưu trữ bền vững bằng PostgreSQL | Truy vấn telemetry và lệnh trước và sau khi làm mới hoặc khởi động lại API |
| Điều khiển quạt/đèn/rèm | ID lệnh đi kèm bằng chứng hoạt động của thiết bị vật lý |
| `Pending` → `Executed` | Phản hồi tạo lệnh và truy vấn sau đó cho cùng một ID lệnh |
| ACK của lệnh | Dòng log nối tiếp của thiết bị, log backend và trạng thái cuối đã lưu |
| Giám sát CloudWatch | Log backend mới, các metric EC2/RDS và cấu hình cảnh báo |

Chỉ đánh dấu hoàn tất khi đã đính kèm bằng chứng tương ứng. Mô hình hiện tại chưa chứng minh HA, độ trễ dưới 50 ms, khả năng “không thể lỗi”, HTTPS, xác thực hoặc điều khiển bằng AI.

## Phát hiện khi rà soát mã nguồn

- Backend chưa kiểm tra lệnh có thuộc danh sách được hỗ trợ hay không; giá trị không hợp lệ vẫn có thể được lưu ở trạng thái `Pending`.
- Cơ chế thăm dò trả bản ghi đang chờ cũ nhất theo FIFO, dù route có tên `commands/latest`.
- Xử lý ACK tìm theo ID lệnh nhưng chưa xác minh ID thiết bị trong route hoặc trạng thái trước đó.
- `DEVICE_API_KEY` có giá trị mặc định trong cấu hình nhưng các route chưa thực sự kiểm tra giá trị này.
- Khi lấy dữ liệu thất bại, frontend có thể chuyển sang dữ liệu `SIMULATED`; một lệnh lỗi vẫn có thể bị hiển thị như trạng thái mô phỏng thành công.
- Chế độ trên frontend chỉ là trạng thái cục bộ, chưa được API xác nhận; các nhãn “AI” và “FAIL-PROOF” mô tả quá mức hành vi dựa trên luật của bản demo.
- Frontend đang gọi giá trị ADC ánh sáng thô là Lux và ghi trực tiếp địa chỉ EC2 trong cấu hình Vite.
- Một phần tài liệu phần cứng trong kho mã nguồn ghi servo ở GPIO 8 và không nhắc LCD, trong khi firmware đang hoạt động dùng GPIO 38 và có LCD1602. Workshop lấy mã đang chạy làm chuẩn.

## Các điều chỉnh riêng của dự án (Project Customizations)

Dự án không sao chép nguyên trạng một bài hướng dẫn có sẵn. Nhóm đã điều chỉnh và đối chiếu các nội dung sau:

- mô hình `room_01` kết nối telemetry vật lý, lịch sử trên dashboard và trạng thái thiết bị chấp hành;
- vòng đời lệnh FastAPI/PostgreSQL lưu `Pending`, `Executed` và ACK từ thiết bị;
- tám lệnh firmware cho chế độ tự động/thủ công cùng khả năng điều khiển trực tiếp quạt, đèn và rèm;
- các ngưỡng điều khiển, sơ đồ GPIO, LCD1602, thời gian kết nối lại và cơ chế khôi phục ACK bằng ESP32 Preferences;
- dashboard React/Vite có biểu đồ telemetry, bảng điều khiển, đề xuất dựa trên luật và cách phân biệt nguồn dữ liệu thật/mô phỏng;
- kết nối RDS riêng thông qua tham chiếu Security Group, không mở cơ sở dữ liệu công khai;
- namespace CloudWatch riêng, cấu hình thu thập log backend và năm alarm đã được ghi nhận; và
- Workshop song ngữ ưu tiên bằng chứng, phân biệt rõ hành vi xác minh từ mã nguồn với kết quả còn cần ảnh chụp hoặc kiểm thử.

Những điều chỉnh trên giúp kiến trúc bám sát mã nguồn và phần cứng YOLO UNO thực tế. Auto Scaling, Amazon SQS, AWS IoT Core và kiến trúc hướng sự kiện vẫn chỉ là lựa chọn trong tương lai, chưa được triển khai.

## Đóng góp cá nhân

| Thành viên | Phạm vi phụ trách và đóng góp cụ thể | Đường dẫn bằng chứng |
| :--- | :--- | :--- |
| **Phạm Lê Minh Khôi** | Kiến trúc AWS, ranh giới mạng/bảo mật, vận hành EC2/RDS/CloudWatch, nối dây YOLO UNO, cảm biến/thiết bị chấp hành, thăm dò telemetry/lệnh, thực thi và ACK | [Kiến trúc](../5.3-Architecture-and-Service-Design/), [thiết lập AWS](../5.4-AWS-Infrastructure-Setup/), [phần cứng](../5.6-Hardware-Integration/), [CloudWatch](../5.9-CloudWatch-Monitoring/) |
| **Ngô Minh Thuận** | Các route FastAPI, alias trong Pydantic, mô hình SQLAlchemy, lưu trữ bền vững bằng PostgreSQL, dịch vụ telemetry, vòng đời lệnh và xử lý ACK | [Thiết kế API/dữ liệu](../5.3-Architecture-and-Service-Design/), [backend/cơ sở dữ liệu](../5.5-Backend-and-Database/), [ma trận kiểm thử](../5.8-End-to-End-Testing/) |
| **Thượng Đình Hưng** | Dashboard React/Vite, trực quan hóa telemetry, yêu cầu điều khiển, giao diện chế độ/đề xuất, gỡ lỗi tích hợp và quay/dựng video minh họa | [tích hợp frontend](../5.7-Frontend-Integration/), [xác minh đầu cuối](../5.8-End-to-End-Testing/), [bàn giao](../5.12-Project-Handover/) |
| **Lê Bảo Khánh** | Nội dung đề xuất/báo cáo, blog/nhật ký/sự kiện, cấu trúc Workshop song ngữ, đối chiếu mã nguồn với tài liệu, điều hướng, kế hoạch ảnh và bảo đảm chất lượng | [tổng quan Workshop](../5.1-Workshop-overview/), [kế hoạch kiểm thử/bằng chứng](../5.8-End-to-End-Testing/), [kết quả](../5.11-Results-Challenges-Future/), [bàn giao](../5.12-Project-Handover/) |

Một đóng góp chỉ được nghiệm thu khi phần liên kết đi kèm mã commit, ảnh chụp, log, kết quả kiểm thử, lịch sử tài liệu hoặc bằng chứng khác giúp xác định người thực hiện. Bảng này chỉ ghi phạm vi phụ trách, không thay thế phần nhìn lại cá nhân bên dưới.

## Nhìn lại theo từng thành viên (Individual Reflections)

### Phạm Lê Minh Khôi

| Nội dung nhìn lại | Trình bày |
| :--- | :--- |
| Challenge | Tích hợp backend phục vụ demo, PostgreSQL trong mạng riêng, hệ thống giám sát và thiết bị chấp hành vật lý mà vẫn phân biệt rõ thành công trên cloud với thành công của phần cứng |
| Root Cause | Luồng xử lý đi qua nhiều lớp: quy tắc VPC, IAM, dịch vụ Linux, cơ chế thăm dò HTTP, đấu nối phần cứng và trạng thái ACK bất đồng bộ |
| Solution | Dùng tham chiếu Security Group từ EC2 tới RDS, gắn EC2 IAM Role, kiểm tra `systemd` và CloudWatch, đối chiếu GPIO với mã nguồn, cấp nguồn an toàn, theo dõi ID lệnh và lưu trạng thái để gửi lại ACK |
| Lesson Learned | Cần xác minh từng ranh giới độc lập và theo dõi cùng một ID lệnh qua API, cơ sở dữ liệu, cổng nối tiếp, thao tác vật lý và hệ thống giám sát |
| Future Improvement | Bổ sung HTTPS/endpoint ổn định, Infrastructure as Code, giới hạn IAM chặt hơn, bằng chứng phần cứng đã hiệu chuẩn và chỉ đánh giá MQTT được quản lý sau khi rà soát kiến trúc |

### Ngô Minh Thuận

| Nội dung nhìn lại | Trình bày |
| :--- | :--- |
| Challenge | Lưu telemetry và giúp người vận hành theo dõi được toàn bộ quá trình hoàn tất lệnh qua cơ chế thăm dò và ACK |
| Root Cause | Các máy khách hoạt động bất đồng bộ nên trạng thái cơ sở dữ liệu có thể lệch; mã nguồn còn thiếu kiểm tra enum cho lệnh và chưa ràng buộc chặt ACK với thiết bị |
| Solution | Mô hình hóa thiết bị, telemetry và lệnh trong PostgreSQL; trả về ID cùng trạng thái lệnh; xử lý lệnh chờ theo FIFO và cập nhật trạng thái rõ ràng khi nhận ACK |
| Lesson Learned | Đặc tả OpenAPI và trạng thái được lưu giúp tăng khả năng truy vết, nhưng kiểm tra đầu vào, phân quyền, tính lũy đẳng và quy trình thay đổi lược đồ phải được thiết kế rõ |
| Future Improvement | Bổ sung kiểm tra lệnh được hỗ trợ, danh tính thiết bị đã xác thực, ACK gắn với thiết bị, quy tắc lũy đẳng, migration Alembic và kiểm thử API tự động |

### Thượng Đình Hưng

| Nội dung nhìn lại | Trình bày |
| :--- | :--- |
| Challenge | Hiển thị telemetry và điều khiển gần thời gian thực, đồng thời phân biệt rõ việc backend nhận yêu cầu, thiết bị thực thi lệnh và giao diện chuyển sang dữ liệu mô phỏng |
| Root Cause | Frontend thăm dò nhiều endpoint, chỉ lưu chế độ trên máy người dùng, chuyển sang dữ liệu mô phỏng khi lỗi và có thể báo thành công dù yêu cầu gửi lệnh thất bại |
| Solution | Kiểm tra thẻ Network của DevTools, dùng route API số nhiều với đường dẫn tương đối, hiển thị ID và trạng thái lệnh, gắn nhãn dữ liệu mô phỏng, đồng thời xác minh thao tác vật lý bằng ACK và bằng chứng |
| Lesson Learned | Giao diện phản hồi nhanh chưa đủ; trạng thái vận hành phải dựa trên backend và thiết bị, còn xử lý lỗi không được ám chỉ thành công khi chưa xác minh |
| Future Improvement | Loại bỏ cơ chế báo thành công giả, bổ sung chế độ/trạng thái lệnh từ API, tập trung cấu hình môi trường, sửa nhãn Lux và thêm kiểm thử component/tích hợp |

### Lê Bảo Khánh

| Nội dung nhìn lại | Trình bày |
| :--- | :--- |
| Challenge | Chuyển các ghi chú mã nguồn thường xuyên thay đổi, đôi khi mâu thuẫn, thành một Workshop song ngữ nhất quán mà không tự suy diễn bằng chứng triển khai |
| Root Cause | Workshop cũ mô tả dịch vụ không liên quan, phần diễn giải khác với firmware đang chạy và ảnh/kết quả kiểm thử bắt buộc chưa đầy đủ |
| Solution | Lấy mã nguồn đang hoạt động làm chuẩn, đồng bộ cấu trúc Anh–Việt, nêu rõ hạn chế, đặt TODO đúng vị trí cho bằng chứng còn thiếu và kiểm tra Hugo, cấu trúc cùng liên kết |
| Lesson Learned | Tài liệu kỹ thuật phải phân biệt rõ nội dung đã triển khai, nội dung đề xuất, kết quả mong đợi và phần đã có bằng chứng; đồng thời giữ lệnh, tên và đường dẫn nhất quán giữa hai ngôn ngữ |
| Future Improvement | Bổ sung CI để kiểm tra Hugo, liên kết, thông tin bí mật và tính đồng bộ Anh–Việt; thay TODO bằng bằng chứng có người chịu trách nhiệm; duy trì đặc tả API/GPIO có phiên bản và lên lịch để các thành viên rà soát, xác nhận |

## Thách thức và bài học

| Vấn đề | Nguyên nhân gốc | Cách xử lý | Bài học |
| :--- | :--- | :--- | :--- |
| Khóa SSH bị từ chối trên Windows | Sai đường dẫn/quyền ACL của khóa hoặc sai tài khoản đăng nhập | Dùng đúng tài khoản của AMI và giới hạn quyền file khóa trên máy cục bộ | Chẩn đoán danh tính trước khi thay đổi quy tắc mạng |
| Sai cú pháp biến môi trường | PowerShell, CMD và Bash dùng cú pháp khác nhau | Chỉ dùng `$env:...`, `%...%`, `$HOME` trong đúng shell | Ghi rõ môi trường thực thi cho mọi lệnh |
| Không truy cập được cổng 8000 | SG đóng hoặc Uvicorn bind vào `127.0.0.1` | Mở đúng nguồn đã duyệt, bind `0.0.0.0` cho demo | Kiểm tra sức khỏe cục bộ trước khi thử đường công khai |
| Kết nối SSL tới RDS lỗi | Sai đường dẫn CA, hostname hoặc `DATABASE_URL` | Dùng gói chứng chỉ hiện hành và đường dẫn tuyệt đối; kiểm tra endpoint | Kết nối mạng và xác minh TLS là hai lớp khác nhau |
| `systemd` khởi động thất bại | Sai tài khoản, đường dẫn, module hoặc môi trường | Xem trạng thái/journal và dùng đúng lệnh đã chạy thủ công thành công | Chỉ đưa lệnh đã xác minh vào unit |
| Vite proxy trả 404/CORS | Sai đích/đường dẫn hoặc yêu cầu đi vòng qua proxy | Dùng `/api` tương đối, khởi động lại Vite, kiểm tra Network | Duy trì một cấu hình gốc API |
| IP công khai thay đổi | EC2 nhận địa chỉ mới sau khi dừng/khởi động | Cập nhật cấu hình trên máy cục bộ và thiết bị | Endpoint ổn định là việc cần làm sau |
| Endpoint không đồng nhất | Route số ít/số nhiều hoặc máy khách dùng đặc tả cũ | Lấy OpenAPI và mã nguồn làm chuẩn | Dùng chung một đặc tả API có phiên bản |
| Lệnh bị trùng | Thăm dò/làm mới/thử lại khiến lệnh được gửi hoặc chạy hai lần | Kiểm tra trạng thái chờ và ID lệnh gần nhất | Gửi lại ACK không được làm thiết bị hoạt động lần nữa |
| ACK quá nhanh làm mất `Pending` | Thiết bị thăm dò và thực thi ngay | Lưu phản hồi tạo lệnh và trạng thái cuối có cùng ID | Bằng chứng phải bám theo cùng một ID |
| Giá trị ánh sáng không chính xác | ADC thô chưa được hiệu chuẩn | Gọi là giá trị analog và hiệu chuẩn sau | Không tự gán đơn vị Lux |
| CloudWatch Agent không có dữ liệu | Sai IAM, đường dẫn, dimension hoặc cấu hình | Xem log Agent và nguồn log thực tế | Quyền truy cập và cấu hình thu thập là hai vấn đề riêng |

## Cải tiến tương lai

- Cân nhắc Elastic IP hoặc domain để có endpoint ổn định.
- Thêm Nginx hoặc reverse proxy đã được rà soát.
- Bổ sung HTTPS, xác thực và phân quyền chặt chẽ hơn.
- Lưu bí mật ứng dụng trong giải pháp quản lý bí mật phù hợp.
- Hỗ trợ nhiều thiết bị/phòng với quy tắc sở hữu và phân quyền.
- Đánh giá WebSocket hoặc MQTT để giảm chi phí thăm dò.
- Đánh giá AWS IoT Core như một lựa chọn truyền thông trong tương lai; hiện chưa triển khai.
- Đóng gói bằng container khi phù hợp và định nghĩa hạ tầng bằng mã.
- Tự động hóa quy trình triển khai và quay lui đã được kiểm thử.
- Bổ sung kênh thông báo cho cảnh báo sau khi rà soát.
- Hiệu chuẩn cảm biến ánh sáng và công bố phương pháp cùng đơn vị quy đổi.

Mỗi hạng mục trong tương lai cần có người phụ trách, được rà soát kiến trúc, phân tích chi phí và bảo mật, triển khai rồi kiểm thử trước khi bổ sung vào tài liệu kiến trúc hiện tại.

## Kết quả

Sau khi rà soát bằng chứng, hãy ghi **Đạt**, **Không đạt** hoặc **Chưa chạy** cho từng điều kiện nghiệm thu. Với mọi nội dung còn thiếu, cần liên kết vấn đề và chỉ định người phụ trách. Không đổi “mong đợi” thành “đã đạt” khi chưa có bằng chứng.

Tiếp theo: [chuẩn bị bàn giao dự án](../5.12-Project-Handover/).
