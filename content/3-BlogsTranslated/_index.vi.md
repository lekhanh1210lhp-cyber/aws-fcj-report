---
title: "Các Blog Đã Dịch"
date: "2025-09-09"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

### [Blog 1 - Cách PostNL xử lý hàng tỷ sự kiện IoT với Amazon Managed Service for Apache Flink](3.1-blog1/)

Bài blog này trình bày chi tiết cách **PostNL** hiện đại hóa nền tảng xử lý luồng dữ liệu IoT của mình bằng cách chuyển đổi sang **Amazon Managed Service for Apache Flink** để xử lý hàng tỷ sự kiện IoT từ các tài sản được theo dõi. Nó nêu bật những thách thức của dữ liệu IoT theo thời gian thực ở quy mô lớn, đặc biệt là việc xử lý các sự kiện đến muộn và ngữ nghĩa thời gian sự kiện (event time semantics). Bằng cách tận dụng **ProcessFunction API** của Flink để kiểm soát ở cấp độ chi tiết, PostNL đã xây dựng thành công một giải pháp xử lý luồng có khả năng mở rộng, mạnh mẽ và tiết kiệm chi phí cho các hoạt động hậu cần của mình.

### [Blog 2 - Tối ưu hóa phân tích dữ liệu IoT công nghiệp với Amazon Data Firehose và Amazon S3 Tables cùng Apache Iceberg](3.2-blog2/)

Bài blog này trình bày cách xây dựng một khuôn khổ thu thập dữ liệu (data ingestion framework) từ biên lên đám mây (edge-to-cloud) dạng low-code, có khả năng mở rộng cho các phân tích **IoT công nghiệp (IIoT)**. Nó giải thích cách thu thập dữ liệu cảm biến thời gian thực bằng cách sử dụng **AWS IoT Greengrass** tại biên, truyền dữ liệu qua **Amazon Data Firehose**, và tối ưu hóa việc lưu trữ bằng cách sử dụng **Amazon S3 Tables** với định dạng **Apache Iceberg**. Kiến trúc này cho phép truy vấn và phân tích hiệu quả, hiệu suất cao và tiết kiệm chi phí thông qua **Amazon Athena** mà không yêu cầu thiết lập cơ sở hạ tầng phức tạp.

### [Blog 3 - Xử lý hàng triệu sự kiện observability với Apache Flink và ghi trực tiếp vào Prometheus](3.3-blog3/)

Bài blog này giới thiệu một **trình kết nối (connector) Apache Flink mới cho Prometheus**, cho phép các ứng dụng Flink ghi trực tiếp dữ liệu chuỗi thời gian đã được tiền xử lý vào **Amazon Managed Service for Prometheus**. Nó khám phá cách việc tiền xử lý các sự kiện observability thô từ các tài sản phân tán cao độ (như thiết bị IoT và ô tô được kết nối) bằng **Amazon Managed Service for Apache Flink** giúp giảm độ đa dạng (cardinality) và tần suất của dữ liệu. Phương pháp này cho phép các tổ chức xây dựng các bảng điều khiển (dashboards) và cảnh báo thời gian thực hiệu quả và có khả năng mở rộng hơn trong **Amazon Managed Grafana**.