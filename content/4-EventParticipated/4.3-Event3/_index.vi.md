---
title: "Sự kiện 3"
date: "2026-07-28"
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Báo cáo về “First Cloud AI Journey: Bảo mật Ứng dụng, Chiến lược thi Cloud Practitioner và Giám sát SLA”

### Mục đích của sự kiện

- Khám phá cách bảo mật các ứng dụng web bằng các tác nhân AI tự trị (autonomous AI agents) với AWS Security Agent (Frontier Agent).
- Tìm hiểu lộ trình chiến lược và các kỹ thuật thực tế để chinh phục kỳ thi chứng chỉ AWS Certified Cloud Practitioner (CLF-C02).
- Hiểu về Thỏa thuận Mức Dịch vụ (SLA) và cách chuyển đổi từ giám sát cơ sở hạ tầng cơ bản sang giám sát lấy người dùng làm trung tâm (user-centric monitoring).

### Diễn giả

- **Thịnh Nguyễn** – Kỹ sư DevOps / DevSecOps / Cloud tại Styl Solutions.
- **Ngô Lê Tấn Huy** – Diễn giả về Chiến lược thi AWS Cloud Practitioner.
- **Nguyễn Huỳnh Sơn** – Kỹ sư Hỗ trợ Cơ sở hạ tầng tại Endava, Thành viên nhóm AWS Student Builder HUFLIT.

### Những Điểm Nổi Bật

## Nội dung chính

1. **Bảo mật Ứng dụng Web với AWS Security Agent**
   - Kiểm thử xâm nhập (pentest) thủ công truyền thống thường tốn thời gian, đắt đỏ ($5k–$20k mỗi lần) và thiếu tính nhất quán.
   - AWS Security Agent (Frontier Agent) sử dụng khả năng suy luận tự trị được cung cấp sức mạnh bởi Amazon Bedrock trên toàn bộ vòng đời ứng dụng: Đánh giá Thiết kế (Design Review), Đánh giá Mã nguồn (Code Review) và Kiểm thử Xâm nhập Chủ động (Active Pentesting).
   - **Đánh giá Thiết kế:** Phân tích các tài liệu kiến trúc hoặc mã Terraform đối chiếu với các framework như PCI DSS, NIST CSF và AWS Well-Architected (Gói miễn phí: 200 lượt đánh giá/tháng).
   - **Đánh giá Mã nguồn:** Tích hợp trực tiếp vào các Pull Request (PR) trên GitHub/GitLab để bình luận về các lỗ hổng, phát hiện dữ liệu nhạy cảm (secrets) và đề xuất các bản sửa lỗi tự động cho PR (Gói miễn phí: 1.000 lượt đánh giá PR/tháng).
   - **Pentest Tự động:** Thực thi các chuỗi khai thác đa bước với các bằng chứng có thể xác minh, mặc dù vẫn tồn tại các hạn chế về chặn xác thực (MFA/sinh trắc học) và các lỗi logic nghiệp vụ.

2. **Lộ trình Chiến lược thi AWS Cloud Practitioner (CLF-C02)**
   - **Tổng quan Kỳ thi:** 65 câu trắc nghiệm, 90 phút (+30 phút cho người không dùng tiếng Anh bản ngữ), điểm đậu 700/1000, có giá trị trong 3 năm.
   - **Tỷ trọng các phần:** Khái niệm Cloud (24%), Bảo mật và Tuân thủ (30%), Công nghệ và Dịch vụ Cloud (34%), và Thanh toán, Đặt giá & Hỗ trợ (12%).
   - **Chiến lược Ôn thi:** Ánh xạ các dịch vụ với các từ khóa (ví dụ: "Decouple/Tách rời" → SQS), xem lại kỹ các lỗi sai khi làm bài thi thử và thực hành trực tiếp bằng AWS Free Tier.
   - **Mẹo & Thủ thuật:** Sử dụng phương pháp loại trừ để bỏ đi các dịch vụ giả hoặc không liên quan, tránh suy nghĩ quá phức tạp ở các câu hỏi nền tảng đơn giản và gắn cờ (flag) các câu hỏi không chắc chắn để xem lại sau.

3. **Từ SLA đến Giám sát những gì thực sự quan trọng**
   - Thỏa thuận Mức Dịch vụ (SLA) xác định các mức dịch vụ kỳ vọng giữa nhà cung cấp và khách hàng nhằm đảm bảo trách nhiệm giải trình và quản lý rủi ro.
   - **Khoảng trống Giám sát (The Monitoring Gap):** Một bảng điều khiển cơ sở hạ tầng "màu xanh" (CPU thấp, health check 200 OK) không đảm bảo trải nghiệm người dùng tốt nếu kết nối cơ sở dữ liệu bị lỗi trong quá trình người dùng đăng nhập.
   - **Kim tự tháp Giám sát:** Mở rộng tầm nhìn xuyên suốt Nhà cung cấp Cloud (Cloud Provider) → Cơ sở hạ tầng → Ứng dụng → Chỉ số Kinh doanh → Trải nghiệm Khách hàng.
   - **Luồng Cảnh báo (Alerting Flow):** Theo dõi các chỉ số ứng dụng tùy chỉnh (ví dụ: tỷ lệ lỗi đăng nhập) trong CloudWatch và gửi cảnh báo sớm qua Amazon SNS đến Slack hoặc email trước khi khách hàng phàn nàn.

### Những Bài học Quan trọng

- **Tư duy Thiết kế & Vận hành**
  - "Mọi thứ đều có thể bị lỗi vào mọi lúc, vì vậy hãy lập kế hoạch cho sự cố và sẽ không có gì thất bại" (Tiến sĩ Werner Vogels, CTO của Amazon).
  - Chỉ riêng các chỉ số cơ sở hạ tầng khỏe mạnh không đồng nghĩa với một trải nghiệm người dùng tốt; việc giám sát phải tập trung vào hành trình khách hàng từ đầu đến cuối (end-to-end).

- **Kiến trúc Kỹ thuật**
  - Các tác nhân AI bảo mật tự trị bổ trợ cho đội ngũ bảo mật con người bằng cách tự động hóa quá trình quét PR, kiểm tra tính tuân thủ của kiến trúc và xác minh lỗ hổng với chi phí chỉ bằng một phần nhỏ.
  - Triển khai giám sát từ trên xuống (top-down) bằng cách theo dõi các chỉ số cấp ứng dụng và kết quả kinh doanh song song với tình trạng sức khỏe của máy chủ.

- **Chiến lược Thi & Học tập**
  - Việc học cho các chứng chỉ nền tảng đòi hỏi phải hiểu các trường hợp sử dụng dịch vụ cốt lõi và loại trừ các câu trả lời gây nhiễu rõ ràng, thay vì phải ghi nhớ các cấu hình phức tạp.
  - Nhận thức được các hạn chế của các công cụ bảo mật tự động, chẳng hạn như rào cản xác thực (MFA, mTLS) và bối cảnh logic nghiệp vụ phức tạp.

### Ứng dụng vào Công việc

- **Trong cơ sở hạ tầng & vận hành**:
  - Chuyển đổi từ việc chỉ giám sát cơ sở hạ tầng thuần túy sang theo dõi các chỉ số lấy người dùng làm trung tâm (ví dụ: tỷ lệ đăng nhập thành công, hoàn tất thanh toán).
  - Cấu hình các CloudWatch Alarms (Cảnh báo) tùy chỉnh kết nối với các chủ đề SNS để nhận thông báo chủ động qua Slack hoặc email khi xảy ra các điểm bất thường trong vận hành.

- **Trong phát triển phần mềm & AI**:
  - Tích hợp các đánh giá bảo mật mã nguồn tự động vào các pipeline CI/CD để phát hiện sớm các thông tin nhạy cảm (secrets) và lỗ hổng mã nguồn ngay từ các Pull Request.
  - Sử dụng các tài nguyên miễn phí như AWS Skill Builder và AWS Free Tier để xây dựng kinh nghiệm thực hành song song với việc chuẩn bị lý thuyết cho chứng chỉ.

- **Trong bảo mật & tuân thủ**:
  - Sử dụng các đánh giá bảo mật thiết kế tự động trên mã Terraform để đảm bảo cơ sở hạ tầng tuân thủ các tiêu chuẩn PCI DSS, NIST CSF và AWS Well-Architected.
  - Áp dụng Mô hình Trách nhiệm Chia sẻ (Shared Responsibility Model) trong các công việc hàng ngày: AWS quản lý bảo mật *của* đám mây, trong khi các nhà phát triển chịu trách nhiệm bảo mật *trong* đám mây và sự hài lòng của khách hàng.

#### Hình ảnh Sự kiện

<img src="/images/4-EventParticipated/image_3.jpg" alt="Event 3" width="600"/>