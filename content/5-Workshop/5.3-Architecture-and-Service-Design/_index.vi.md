---
title: "Kiến trúc và thiết kế dịch vụ"
date: "2026-07-28"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Kiến trúc

![Kiến trúc AWS IoT Monitoring and Control Dashboard](/images/5-Workshop/5.3-architecture/aws-iot-dashboard-architecture.png)

*Hình 5-2. Sơ đồ được sao chép từ kho mã nguồn ứng dụng, thể hiện ranh giới dịch vụ, kết nối EC2-RDS, các đường giao tiếp HTTP của thiết bị và luồng giám sát CloudWatch.*

Người dùng dashboard, React + Vite frontend chạy trên máy cục bộ và YOLO UNO nằm ngoài AWS. Trong AWS Cloud, VPC chứa public subnet dành cho EC2 và DB Subnet Group dành cho RDS. EBS là ổ đĩa gốc của EC2. Hai Security Group của EC2 và RDS kiểm soát lưu lượng mạng. IAM Role thuộc phạm vi tài khoản AWS, còn CloudWatch là dịch vụ theo khu vực; cả hai đều không nằm bên trong VPC.

## Thành phần và lý do chọn dịch vụ AWS

| Thành phần/dịch vụ | Trách nhiệm và lý do |
| :--- | :--- |
| React + Vite + TypeScript + Tailwind CSS | Giao diện cục bộ cho người vận hành, hiển thị telemetry, điều khiển và đưa ra đề xuất dựa trên luật |
| Amazon EC2 | Toàn quyền cấu hình FastAPI, Python, Uvicorn và `systemd` |
| Amazon EBS | Ổ đĩa gốc lưu trữ bền vững, gắn với EC2 |
| Amazon RDS for PostgreSQL | Lưu telemetry và trạng thái lệnh theo mô hình quan hệ |
| Amazon VPC và subnet | Ranh giới mạng cho EC2 và DB Subnet Group |
| Security Group | Các quy tắc có trạng thái cho SSH/API và lưu lượng từ EC2 tới RDS |
| AWS IAM Role | Cấp quyền tạm thời để EC2 gửi dữ liệu giám sát |
| CloudWatch Agent | Phần mềm trên EC2 thu thập metric của hệ điều hành khách và file log |
| Amazon CloudWatch/Alarms | Lưu metric/log và đánh giá các ngưỡng |
| YOLO UNO / ESP32-S3 | Đọc cảm biến, điều khiển thiết bị chấp hành, thăm dò lệnh và gửi ACK |

IAM Role có nhiệm vụ cấp quyền; nó không phải CloudWatch Agent. Agent là một tiến trình được cài đặt và quản lý trên EC2.

## 5.3.3 Lựa chọn dịch vụ AWS và các đánh đổi kiến trúc

Dự án lựa chọn các dịch vụ AWS dựa trên bốn tiêu chí chính:

1. Phù hợp với kiến trúc và mã nguồn hiện tại.
2. Đơn giản để triển khai và giải thích trong Workshop.
3. Có thể giám sát, kiểm thử và vận hành trực tiếp.
4. Có chi phí hợp lý cho môi trường học tập và trình diễn.

Không phải dịch vụ serverless nào cũng cần thiết cho trường hợp sử dụng này. Hệ thống hiện tại chạy FastAPI liên tục, kết nối PostgreSQL và giao tiếp với YOLO UNO qua REST API sử dụng HTTP. Vì vậy, nhóm chọn Amazon EC2 và Amazon RDS thay vì thiết kế lại toàn bộ hệ thống theo Lambda, API Gateway và DynamoDB.

### Các dịch vụ được lựa chọn

| Dịch vụ AWS | Vai trò trong hệ thống | Lý do lựa chọn | Đánh đổi |
| :--- | :--- | :--- | :--- |
| **Amazon EC2** | Chạy backend FastAPI, Uvicorn và CloudWatch Agent | Cho phép chủ động cấu hình môi trường Python, thư viện phụ thuộc, cổng mạng, dịch vụ `systemd` và file log | Nhóm phải tự quản lý hệ điều hành, cập nhật gói phần mềm, dịch vụ và một phần cấu hình bảo mật |
| **Amazon EBS** | Ổ đĩa gốc của EC2 | Cung cấp nơi lưu trữ bền vững cho hệ điều hành, mã nguồn, môi trường ảo và log cục bộ | Ổ đĩa không còn gắn vẫn có thể phát sinh chi phí nếu không được xóa |
| **Amazon RDS for PostgreSQL** | Lưu telemetry và trạng thái lệnh | PostgreSQL phù hợp với dữ liệu có cấu trúc và quan hệ giữa thiết bị, telemetry, lệnh; RDS giảm công việc quản trị so với tự cài PostgreSQL trên EC2 | Một cơ sở dữ liệu chạy liên tục có thể tốn hơn một số giải pháp serverless khi lưu lượng rất thấp |
| **Amazon VPC** | Cung cấp môi trường mạng cho EC2 và RDS | Cho phép kiểm soát subnet, định tuyến và kết nối giữa backend với cơ sở dữ liệu | Đòi hỏi cấu hình mạng và Security Group chính xác |
| **Security Groups** | Kiểm soát lưu lượng vào EC2 và RDS | Giới hạn SSH theo IP quản trị và chỉ cho phép RDS nhận cổng 5432 từ EC2 Security Group | Cấu hình sai có thể chặn kết nối cơ sở dữ liệu hoặc vô tình mở dịch vụ quá rộng |
| **AWS IAM Role** | Cấp quyền để EC2 gửi log và metric | Tránh lưu AWS access key trực tiếp trên EC2 hoặc trong mã nguồn | Policy phải tuân theo nguyên tắc đặc quyền tối thiểu |
| **Amazon CloudWatch** | Thu thập log và metric từ EC2, backend, RDS | Tập trung dữ liệu vận hành để quan sát và xử lý sự cố | Việc thu nhận log, thời gian lưu giữ và metric tùy chỉnh có thể phát sinh chi phí |
| **CloudWatch Alarms** | Theo dõi CPU, bộ nhớ, ổ đĩa và số kết nối cơ sở dữ liệu | Cảnh báo khi metric vượt ngưỡng vận hành đã cấu hình | Alarm chỉ hữu ích khi ngưỡng và khoảng thời gian đánh giá được đặt phù hợp |

### Vì sao chọn Amazon EC2 cho FastAPI backend?

FastAPI backend của dự án là một ứng dụng chạy liên tục và cung cấp nhiều REST API cho frontend và thiết bị YOLO UNO. Amazon EC2 phù hợp vì nhóm có thể:

- Cài đặt phiên bản Python và các thư viện phụ thuộc cần thiết.
- Chạy Uvicorn dưới dạng dịch vụ `systemd`.
- Chủ động cấu hình cổng `8000`.
- Cài CloudWatch Agent.
- Truy cập log và kiểm tra trạng thái dịch vụ qua SSH.
- Kết nối trực tiếp đến Amazon RDS for PostgreSQL.
- Gỡ lỗi toàn bộ yêu cầu telemetry, lệnh và ACK.

Amazon EC2 cũng giúp Workshop dễ trình bày hơn vì người học có thể quan sát rõ quá trình cài đặt backend, khởi động service, kiểm tra log và xử lý lỗi.

Đánh đổi chính là EC2 không phải dịch vụ được quản lý hoàn toàn. Nhóm vẫn phải cập nhật hệ điều hành, quản lý dịch vụ, kiểm tra dung lượng lưu trữ và bảo vệ SSH cũng như cổng backend.

### Vì sao chọn Amazon RDS for PostgreSQL?

Dữ liệu của dự án có cấu trúc và quan hệ rõ ràng:

- Một thiết bị gửi nhiều bản ghi telemetry.
- Một thiết bị có thể nhận nhiều lệnh.
- Mỗi lệnh có trạng thái như `Pending` hoặc `Executed`.
- Backend cần truy vấn dữ liệu theo thời gian và ID thiết bị.

PostgreSQL phù hợp với các truy vấn có cấu trúc, việc kiểm tra trạng thái lệnh và lưu lịch sử telemetry. Amazon RDS giảm công việc quản trị so với tự cài PostgreSQL trên EC2, từ khởi tạo cơ sở dữ liệu, quản lý dung lượng đến cung cấp metric tích hợp với CloudWatch.

Đánh đổi là RDS instance vẫn phát sinh chi phí theo thời gian hoạt động, kể cả khi lưu lượng của Workshop thấp.

### Vì sao chọn Amazon EBS, VPC và IAM Role?

- **Amazon EBS** cung cấp ổ đĩa gốc cho EC2, lưu bền vững hệ điều hành, mã nguồn đã lấy về, môi trường ảo Python và log backend cục bộ qua các lần khởi động lại thông thường. Dung lượng, snapshot, mã hóa và chi phí của ổ đĩa không còn gắn vẫn là trách nhiệm vận hành.
- **Amazon VPC** tạo ranh giới rõ ràng bằng subnet, route và Security Group. Thiết kế cho phép truy cập EC2 phục vụ demo trong khi RDS vẫn nằm riêng trong DB Subnet Group. Đổi lại, nhóm phải cấu hình cẩn thận và lưu bằng chứng đầy đủ.
- **AWS IAM Role** cấp thông tin xác thực tạm thời để EC2 gửi dữ liệu lên CloudWatch mà không lưu AWS access key trong mã nguồn hoặc `.env`. Role phải được giới hạn theo hành động và tài nguyên cần thiết; role không thay thế kiểm soát mạng hoặc CloudWatch Agent.

### Vì sao chọn Amazon CloudWatch?

CloudWatch được lựa chọn vì có thể tập trung:

- EC2 `CPUUtilization`.
- Memory và disk metric từ CloudWatch Agent.
- FastAPI backend logs.
- RDS `CPUUtilization`.
- RDS `DatabaseConnections`.
- Trạng thái của CloudWatch Alarms.

Nhờ đó, nhóm có thể chứng minh hệ thống không chỉ hoạt động đúng chức năng mà còn có khả năng giám sát và hỗ trợ xử lý sự cố.

### Các dịch vụ không được sử dụng trong phiên bản hiện tại

| Dịch vụ | Lý do chưa lựa chọn |
| :--- | :--- |
| **AWS Lambda** | Backend FastAPI hiện chạy liên tục dưới dạng dịch vụ trên EC2. Chuyển sang Lambda đòi hỏi thay đổi mô hình triển khai, vòng đời xử lý và cách kết nối cơ sở dữ liệu |
| **Amazon API Gateway** | Frontend và YOLO UNO hiện gọi trực tiếp REST API trên EC2. API Gateway sẽ thêm một lớp dịch vụ và chi phí mà mô hình hiện tại chưa cần |
| **Amazon DynamoDB** | Dữ liệu được thiết kế theo mô hình quan hệ; backend đã dùng SQLAlchemy với PostgreSQL |
| **Amazon S3** | Dự án chưa cần lưu object, tải file lên hoặc phân phối frontend tĩnh từ AWS. Website Workshop được triển khai riêng, không thuộc hệ thống IoT dashboard khi chạy |
| **AWS IoT Core** | YOLO UNO hiện giao tiếp trực tiếp với FastAPI bằng REST API qua HTTP. MQTT và chứng chỉ thiết bị là các lựa chọn có thể xem xét sau này |
| **Amazon SQS** | Luồng lệnh hiện dùng bản ghi `Pending` trong PostgreSQL và cơ chế thăm dò của thiết bị; chưa triển khai hàng đợi, bên gửi hoặc bên nhận SQS |

Việc không sử dụng các dịch vụ trên không có nghĩa chúng không phù hợp với IoT. Đây là quyết định giới hạn phạm vi để nhóm tập trung vào luồng end-to-end giữa phần cứng, REST API, PostgreSQL, dashboard và CloudWatch.

### Đánh giá về chi phí và độ đơn giản

Kiến trúc hiện tại ưu tiên khả năng quan sát và triển khai trực tiếp hơn là tối ưu hoàn toàn theo mô hình serverless.

- **Độ đơn giản:** EC2 cho phép chạy nguyên backend FastAPI mà không phải tách thành nhiều hàm Lambda.
- **Dịch vụ được quản lý:** RDS giảm công việc quản trị so với tự cài PostgreSQL trên EC2.
- **Chi phí:** EC2 và RDS tính phí theo thời gian hoạt động, vì vậy cần dừng hoặc xóa tài nguyên sau Workshop.
- **Giá trị học tập:** Người học có thể thực hành Linux, `systemd`, REST API, PostgreSQL, IAM, Security Group và CloudWatch trong cùng một dự án.
- **Khả năng mở rộng:** Kiến trúc có thể phục vụ thêm một số thiết bị, nhưng khi mở rộng cần bổ sung xác thực, HTTPS, cân bằng tải hoặc kiến trúc hướng sự kiện.

## Đặc tả API đã được xác minh

Mã nguồn FastAPI trong `backend/main.py` và `backend/app/api/` xác nhận các route sau:

| Method | Route | Thành phần gọi |
| :--- | :--- | :--- |
| `GET` | `/` | Thông tin dịch vụ cơ bản |
| `GET` | `/api/health` | Health check |
| `POST` | `/api/telemetry` | Telemetry từ YOLO UNO |
| `GET` | `/api/devices/{device_id}/latest` | Màn hình latest của dashboard |
| `GET` | `/api/devices/{device_id}/history` | Màn hình history của dashboard |
| `POST` | `/api/devices/{device_id}/commands` | Lệnh điều khiển từ dashboard |
| `GET` | `/api/devices/{device_id}/commands/latest` | Thiết bị thăm dò lệnh |
| `POST` | `/api/devices/{device_id}/commands/{command_id}/ack` | ACK từ thiết bị |

Firmware hỗ trợ `MODE_AUTO`, `MODE_MANUAL`, `FAN_ON`, `FAN_OFF`, `LIGHT_ON`, `LIGHT_OFF`, `CURTAIN_OPEN`, `CURTAIN_CLOSE`. Backend hiện chấp nhận mọi chuỗi lệnh vì `DeviceCommand` chưa có bộ kiểm tra enum đang hoạt động; lệnh không được hỗ trợ sẽ giữ trạng thái `Pending` do firmware từ chối và không gửi ACK. Không dùng route số ít `/api/device/...`.

## Luồng dữ liệu

1. **Telemetry:** YOLO UNO gửi các trường camelCase → alias Pydantic ánh xạ sang snake_case → SQLAlchemy ghi vào `telemetry_logs` trên RDS → API mới nhất/lịch sử → dashboard.
2. **Lệnh:** dashboard tạo lệnh → backend ghi `commands.state = "Pending"` → route `commands/latest` thực tế trả lệnh chờ cũ nhất trước (FIFO) → phần cứng thực thi.
3. **ACK:** thiết bị gửi ID lệnh → backend chuyển lệnh đó sang `Executed` → telemetry tiếp theo phản ánh trạng thái thiết bị chấp hành. Dịch vụ ACK hiện chỉ tìm theo ID lệnh; kiểm tra lệnh có thuộc đúng thiết bị hay không vẫn là điểm cần gia cố.
4. **Giám sát:** metric mặc định của EC2 cùng metric/log do agent thu thập được gửi tới CloudWatch; RDS cung cấp metric dịch vụ; alarm đánh giá các ngưỡng đã cấu hình.

Do thiết bị thăm dò lệnh thường xuyên, trạng thái `Pending` có thể chỉ xuất hiện trong thời gian rất ngắn. Bằng chứng nên gồm phản hồi khi tạo lệnh và trạng thái `Executed` sau đó trong cơ sở dữ liệu hoặc API.

Các mô hình cơ sở dữ liệu định nghĩa ba bảng `devices`, `telemetry_logs` và `commands`. Các trường telemetry gồm `temperature`, `humidity`, `light_intensity`, `fan_status`, `light_status`, `curtain_status` và `timestamp`.

## Bảo mật và IAM

### Bảng bảo mật mạng

| Nguồn | Đích | Port | Rule |
| :--- | :--- | :---: | :--- |
| `<ADMIN_IP>/32` | EC2 Security Group | 22 | Chỉ quản trị SSH |
| Máy khách demo | EC2 Security Group | 8000 | API HTTP phục vụ demo; không giữ `0.0.0.0/0` sau buổi demo |
| EC2 Security Group | RDS Security Group | 5432 | Chỉ PostgreSQL |
| EC2/RDS | CloudWatch | HTTPS | Luồng outbound giám sát |

RDS không được đặt ở chế độ công khai. Thông tin bí mật nằm trong các file cục bộ đã được loại khỏi Git; EC2 dùng IAM Role thay cho AWS key viết trực tiếp trong mã nguồn. Thiết kế hiện tại chưa có HTTPS, xác thực, HA, bằng chứng Multi-AZ, cân bằng tải hoặc giới hạn tần suất.

### Bảng bảo mật và IAM

| Kiểm soát | Cách triển khai hiện tại | Bằng chứng cần giữ | Hạn chế / bước gia cố tiếp theo |
| :--- | :--- | :--- | :--- |
| Truy cập quản trị | SSH cổng 22 giới hạn theo `<ADMIN_IP>/32` | Quy tắc EC2 Security Group | Rà soát người giữ khóa, thay khóa khi cần; cân nhắc hình thức truy cập được quản lý |
| Cô lập cơ sở dữ liệu | RDS private; cổng 5432 chỉ nhận EC2 Security Group | Cấu hình RDS, subnet group và tham chiếu SG | Rà soát NACL/route và xác minh TLS |
| Thông tin xác thực AWS | EC2 IAM Role có `CloudWatchAgentServerPolicy` phục vụ giám sát | Instance profile và policy đã gắn | Khi phù hợp, thay policy quản lý rộng bằng policy giới hạn tài nguyên đã được rà soát |
| Bí mật ứng dụng | `.env` và `hardware/include/secrets.h` chỉ lưu cục bộ, đã loại khỏi Git | Vị trí file đã che và kết quả `git status` | Chuyển bí mật dùng trong thực tế sang dịch vụ quản lý bí mật đã được phê duyệt |
| API công khai | HTTP cổng 8000 dùng trực tiếp trong buổi demo có giám sát | EC2 SG và yêu cầu kiểm tra sức khỏe | Bổ sung HTTPS, xác thực, phân quyền và giới hạn tần suất trước khi dùng thực tế |
| Danh tính cơ sở dữ liệu | Người dùng PostgreSQL riêng trong `DATABASE_URL` | Cấu hình kết nối đã che | Giới hạn quyền, thay mật khẩu định kỳ và kiểm toán truy cập |

### Nguyên tắc đặc quyền tối thiểu (Principle of Least Privilege)

Chỉ cấp các hành động cần thiết cho từng danh tính, giới hạn nguồn/đích mạng và tránh dùng thông tin xác thực dài hạn. Workshop không yêu cầu `AdministratorAccess`: người vận hành chỉ cần quyền cấp phát và kiểm tra đã được duyệt; EC2 chỉ cần quyền gửi dữ liệu giám sát; RDS chỉ nhận kết nối PostgreSQL từ EC2 Security Group; người dùng cơ sở dữ liệu chỉ có các quyền ứng dụng thực sự cần. Mọi quyền rộng được cấp tạm để xử lý sự cố phải được ghi lại, giới hạn thời gian, rà soát và gỡ bỏ.

## Mô hình vận hành hiện tại

- Một EC2 Amazon Linux chạy FastAPI/Uvicorn liên tục dưới dịch vụ `aws-iot-backend` của `systemd`.
- Một ổ đĩa gốc EBS lưu hệ điều hành, mã nguồn, môi trường ảo và các file log cục bộ.
- Một RDS for PostgreSQL private lưu thiết bị, telemetry và trạng thái lệnh.
- Dashboard React/Vite chạy cục bộ và một YOLO UNO gọi REST API trên EC2 theo cơ chế thăm dò HTTP định kỳ.
- CloudWatch Agent gửi metric của hệ điều hành khách và hai file log backend; metric gốc của EC2/RDS cùng sáu alarm đã tài liệu hóa hỗ trợ vận hành.
- Việc triển khai, quyết định mở rộng, khôi phục và dọn dẹp hiện được thực hiện thủ công theo tài liệu hướng dẫn.

## Lựa chọn mở rộng tương lai và hạn chế hiện tại

Cấu trúc API route theo `device_id` và lược đồ quan hệ có thể hỗ trợ thêm phòng, nhưng phạm vi nghiệm thu hiện tại chỉ là `room_01`. Một endpoint EC2 kết hợp với cơ chế thăm dò HTTP định kỳ giúp mô hình đơn giản, nhưng có rủi ro địa chỉ IP công khai thay đổi, độ trễ do chu kỳ thăm dò và điểm lỗi đơn tại tầng tính toán.

Các lựa chọn trong tương lai có thể gồm DNS/HTTPS ổn định, xác thực và phân quyền theo thiết bị, Load Balancer với nhiều backend không lưu trạng thái, Auto Scaling, MQTT được quản lý qua AWS IoT Core, xử lý qua hàng đợi như SQS, bộ nhớ đệm, bản sao chỉ đọc, cơ sở dữ liệu Multi-AZ, container và Infrastructure as Code. Mỗi lựa chọn đều cần kiến trúc mới, rà soát chi phí/bảo mật, triển khai và kiểm thử.

**Auto Scaling, Amazon SQS và kiến trúc hướng sự kiện chưa được triển khai trong dự án hiện tại.** Đây chỉ là các lựa chọn mở rộng trong tương lai.

## Kết quả mong đợi và xử lý sự cố

Mỗi mũi tên trong kiến trúc phải tương ứng với một lời gọi API, quy tắc mạng, thao tác cơ sở dữ liệu hoặc đường đi của metric/log. Nếu một kết nối chưa rõ, hãy xác định nguồn, đích, cổng, danh tính và bằng chứng mong đợi trước khi cấp phát tài nguyên.

Tiếp theo: [xây dựng hạ tầng AWS](../5.4-AWS-Infrastructure-Setup/).
