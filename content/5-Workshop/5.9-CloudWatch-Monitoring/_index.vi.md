---
title: "Giám sát với CloudWatch"
date: "2026-07-28"
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Tổng quan và mục tiêu

Sử dụng metric mặc định của EC2/RDS, metric của hệ điều hành khách và log backend do CloudWatch Agent thu thập. EC2 IAM Role cấp quyền gửi dữ liệu, còn CloudWatch Agent là phần mềm chạy riêng trên instance.

## Danh mục giám sát

| Nguồn | Metric/log | Cách thu thập |
| :--- | :--- | :--- |
| EC2 | `CPUUtilization` | Metric mặc định EC2 |
| Guest OS EC2 | `mem_used_percent` | CloudWatch Agent |
| Guest OS EC2 | `disk_used_percent` | CloudWatch Agent |
| Guest OS EC2 | `cpu_usage_idle`, `cpu_usage_user`, `cpu_usage_system` | CloudWatch Agent |
| FastAPI | Log ứng dụng backend | CloudWatch Agent đọc file log |
| RDS | `CPUUtilization` | Metric mặc định RDS |
| RDS | `DatabaseConnections` | Metric mặc định RDS |

## Bước 1 - Xác minh IAM Role và cài Agent

Trong EC2 Console, xác nhận instance đã gắn IAM instance profile có `CloudWatchAgentServerPolicy` được phê duyệt. Trên EC2 Linux Bash, cài gói `amazon-cloudwatch-agent` theo hướng dẫn chính thức cho bản phân phối Linux đã chọn, rồi kiểm tra:

```bash
sudo systemctl status amazon-cloudwatch-agent --no-pager
ls -l /opt/aws/amazon-cloudwatch-agent/bin/
```

Không lưu AWS access key trong cấu hình của Agent.

## Bước 2 - Cấu hình metric và backend log

Tạo `/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json`:

```json
{
  "agent": {
    "metrics_collection_interval": 60,
    "run_as_user": "root"
  },
  "metrics": {
    "namespace": "IoTDashboard/EC2",
    "append_dimensions": {
      "InstanceId": "${aws:InstanceId}"
    },
    "metrics_collected": {
      "mem": {
        "measurement": ["mem_used_percent"]
      },
      "disk": {
        "measurement": ["used_percent"],
        "resources": ["/"]
      },
      "cpu": {
        "measurement": ["cpu_usage_idle", "cpu_usage_user", "cpu_usage_system"],
        "totalcpu": true
      }
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/aws-iot-backend/backend.log",
            "log_group_name": "/aws/ec2/iot-dashboard/backend",
            "log_stream_name": "{instance_id}/backend",
            "timezone": "UTC"
          },
          {
            "file_path": "/var/log/aws-iot-backend/backend-error.log",
            "log_group_name": "/aws/ec2/iot-dashboard/backend-error",
            "log_stream_name": "{instance_id}/backend-error",
            "timezone": "UTC"
          }
        ]
      }
    }
  }
}
```

Nếu dịch vụ chỉ ghi log vào journald, hãy cấu hình ghi log ra file theo mã nguồn hoặc dùng phương thức thu thập journald đã được phê duyệt; không cấu hình Agent đọc một file không tồn tại.

## Bước 3 - Khởi động và bật Agent cùng hệ thống

Trong EC2 Linux Bash:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
  -s
sudo systemctl enable amazon-cloudwatch-agent
sudo systemctl status amazon-cloudwatch-agent --no-pager
```

Xem log chẩn đoán của Agent:

```bash
sudo tail -n 100 /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

## Bước 4 - Tạo và kiểm tra bằng chứng

1. Gọi `/api/health` và gửi một yêu cầu telemetry hợp lệ.
2. Mở CloudWatch trong cùng khu vực.
3. Kiểm tra log group `/aws/ec2/iot-dashboard/backend` và `/aws/ec2/iot-dashboard/backend-error`.
4. Mở **Metrics → IoTDashboard/EC2** để xem bộ nhớ, ổ đĩa và CPU của hệ điều hành khách.
5. Mở **Metrics → EC2** cho `CPUUtilization`.
6. Mở **Metrics → RDS** cho `CPUUtilization` và `DatabaseConnections`.
7. Chọn khoảng thời gian phù hợp và xác nhận có dữ liệu với timestamp mới.

## Bước 5 - Tạo và xác minh các alarm

Tài liệu trong mã nguồn đề xuất chính xác bộ alarm dưới đây. Chỉ xem đây là cấu hình mục tiêu cho đến khi ảnh chụp hoặc dữ liệu xuất ra chứng minh rằng alarm đã được triển khai:

| Tên alarm | Metric | Điều kiện trong tài liệu mã nguồn |
| :--- | :--- | :--- |
| `iot-dashboard-ec2-high-cpu` | `CPUUtilization` | ≥80% trong 5 phút |
| `iot-dashboard-ec2-high-memory` | `mem_used_percent` | ≥80% trong 5 phút |
| `iot-dashboard-ec2-high-disk` | `disk_used_percent` | ≥80% trong 5 phút |
| `iot-dashboard-rds-high-cpu` | `CPUUtilization` | ≥80% trong 5 phút |
| `iot-dashboard-rds-high-connections` | `DatabaseConnections` | ≥10 trong 5 phút |
| `iot-dashboard-ec2-status-check` | `StatusCheckFailed` | ≥1 trong 5 phút |

Hãy xác minh ngưỡng, chu kỳ, số lần đánh giá, cách xử lý dữ liệu thiếu và hành động thực tế, thay vì mặc định rằng tài liệu đã được áp dụng. README gốc nói rõ dự án không dùng SNS; README của backend chỉ nhắc SNS như một tùy chọn mở rộng, vì vậy không được tuyên bố đã triển khai topic hoặc subscription SNS.

- **OK:** các điểm dữ liệu gần đây không vượt quy tắc đã đặt.
- **In alarm:** có đủ điểm dữ liệu vượt ngưỡng đã cấu hình.
- **Insufficient data:** alarm chưa có đủ điểm dữ liệu để đánh giá; trạng thái này không đồng nghĩa hệ thống đang khỏe.

## Kết quả mong đợi

CloudWatch hiển thị sự kiện mới trong cả hai log group của backend, metric bộ nhớ/ổ đĩa/CPU trong `IoTDashboard/EC2`, metric gốc của EC2/RDS và sáu cấu hình alarm có trạng thái giải thích được. Bằng chứng phải phản ánh đúng phần đã triển khai và không tuyên bố có thông báo SNS.

<!-- TODO IMAGE: /images/5-Workshop/5.9-cloudwatch/backend-cloudwatch-logs.png — Hai log group backend và backend-error cùng instance stream mới; che account ID, instance ID, IP và giá trị log nhạy cảm. -->
<!-- TODO IMAGE: /images/5-Workshop/5.9-cloudwatch/ec2-rds-metrics.png — Guest metric IoTDashboard/EC2 cùng graph metric native EC2 và RDS trong cùng time range gần đây. -->
<!-- TODO IMAGE: /images/5-Workshop/5.9-cloudwatch/cloudwatch-alarms.png — Sáu alarm đã tài liệu hóa với tên, threshold và current state; che định danh tài khoản. -->

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| Agent không hoạt động | Cú pháp JSON, log dịch vụ và quá trình cài gói |
| Bị từ chối quyền | Instance profile và policy đã gắn; không dùng AWS key cục bộ |
| Không có metric bộ nhớ/ổ đĩa | Namespace `IoTDashboard/EC2`, dimension, chu kỳ thu thập và việc nạp lại cấu hình |
| Không có log backend | Đường dẫn log thực tế, quyền đọc, yêu cầu mới và timestamp của luồng |
| Alarm thiếu dữ liệu | Sai metric/dimension/khu vực hoặc không có điểm dữ liệu mới |
| Thiếu metric RDS | Đúng DB identifier, khu vực và khoảng thời gian của biểu đồ |

Dự án không sử dụng AI Operations, GenAI Observability, Application Signals, tính năng khám phá tài nguyên hoặc quy trình quan sát nâng cao.

Tiếp theo: [rà soát chi phí, bảo mật và dọn dẹp](../5.10-Cost-Security-Cleanup/).
