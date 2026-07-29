---
title: "Giám sát với CloudWatch"
date: "2026-07-28"
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Tổng quan và mục tiêu

Dùng metric mặc định của EC2/RDS cùng guest metric và backend log từ CloudWatch Agent. EC2 IAM Role cấp quyền gửi dữ liệu; CloudWatch Agent là phần mềm riêng chạy trên instance.

## Danh mục giám sát

| Nguồn | Metric/log | Đường thu thập |
| :--- | :--- | :--- |
| EC2 | `CPUUtilization` | Metric mặc định EC2 |
| Guest OS EC2 | `mem_used_percent` | CloudWatch Agent |
| Guest OS EC2 | `disk_used_percent` | CloudWatch Agent |
| Guest OS EC2 | `cpu_usage_idle`, `cpu_usage_user`, `cpu_usage_system` | CloudWatch Agent |
| FastAPI | Backend application log | Thu log bằng CloudWatch Agent |
| RDS | `CPUUtilization` | Metric mặc định RDS |
| RDS | `DatabaseConnections` | Metric mặc định RDS |

## Bước 1 - Xác minh role và cài agent

Trong EC2 Console, xác nhận instance có IAM instance profile với `CloudWatchAgentServerPolicy` đã duyệt. Trên EC2 Linux Bash, cài package `amazon-cloudwatch-agent` theo quy trình chính thức cho distribution đã chọn, rồi kiểm tra:

```bash
sudo systemctl status amazon-cloudwatch-agent --no-pager
ls -l /opt/aws/amazon-cloudwatch-agent/bin/
```

Không lưu AWS access key trong cấu hình agent.

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

Nếu service đang chạy chỉ log vào journald, hãy cấu hình file logger theo source hoặc dùng cách thu journald đã duyệt; không trỏ agent vào file không tồn tại.

## Bước 3 - Start và enable agent

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

Xem log chẩn đoán agent:

```bash
sudo tail -n 100 /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

## Bước 4 - Tạo và xem evidence

1. Gọi `/api/health` và gửi một telemetry request hợp lệ.
2. Mở CloudWatch trong cùng region.
3. Kiểm tra log group `/aws/ec2/iot-dashboard/backend` và `/aws/ec2/iot-dashboard/backend-error`.
4. Mở **Metrics → IoTDashboard/EC2** cho guest memory/disk/CPU.
5. Mở **Metrics → EC2** cho `CPUUtilization`.
6. Mở **Metrics → RDS** cho `CPUUtilization` và `DatabaseConnections`.
7. Đặt time range phù hợp và xác nhận timestamp mới.

## Bước 5 - Tạo và xác minh alarm

Runbook source đề xuất chính xác bộ alarm dưới đây. Xem đây là target configuration được tài liệu hóa đến khi screenshot/export chứng minh đã deploy:

| Tên alarm | Metric | Điều kiện trong tài liệu source |
| :--- | :--- | :--- |
| `iot-dashboard-ec2-high-cpu` | `CPUUtilization` | ≥80% trong 5 phút |
| `iot-dashboard-ec2-high-memory` | `mem_used_percent` | ≥80% trong 5 phút |
| `iot-dashboard-ec2-high-disk` | `disk_used_percent` | ≥80% trong 5 phút |
| `iot-dashboard-rds-high-cpu` | `CPUUtilization` | ≥80% trong 5 phút |
| `iot-dashboard-rds-high-connections` | `DatabaseConnections` | ≥10 trong 5 phút |
| `iot-dashboard-ec2-status-check` | `StatusCheckFailed` | ≥1 trong 5 phút |

Xác minh threshold, period, evaluation count, missing-data behavior và action đã deploy thay vì giả định runbook đã được áp dụng. README source gốc nói rõ không dùng SNS; backend README chỉ nhắc SNS như tùy chọn mở rộng, vì vậy không tuyên bố topic/subscription SNS đã deploy.

- **OK:** datapoint gần đây không vi phạm rule.
- **In alarm:** đủ datapoint vi phạm threshold đã cấu hình.
- **Insufficient data:** alarm thiếu datapoint dùng được; không đồng nghĩa hệ thống khỏe.

<!-- TODO IMAGE: /images/5-Workshop/5.9-cloudwatch/backend-cloudwatch-logs.png — Hai log group backend và backend-error cùng instance stream mới; che account ID, instance ID, IP và giá trị log nhạy cảm. -->
<!-- TODO IMAGE: /images/5-Workshop/5.9-cloudwatch/ec2-rds-metrics.png — Guest metric IoTDashboard/EC2 cùng graph metric native EC2 và RDS trong cùng time range gần đây. -->
<!-- TODO IMAGE: /images/5-Workshop/5.9-cloudwatch/cloudwatch-alarms.png — Sáu alarm đã tài liệu hóa với tên, threshold và current state; che định danh tài khoản. -->

## Xử lý sự cố

| Hiện tượng | Nội dung cần kiểm tra |
| :--- | :--- |
| Agent inactive | Cú pháp JSON, service log, package installation |
| Access denied | Instance profile và policy đã gắn; không dùng AWS key local |
| Không có memory/disk metric | Namespace `IoTDashboard/EC2`, dimension, interval, reload config |
| Không có backend log | Log path thật, quyền đọc, request mới, timestamp stream |
| Alarm insufficient data | Sai metric/dimension/region hoặc không có datapoint mới |
| Thiếu RDS metric | DB identifier, region và graph time range |

Project không dùng AI Operations, GenAI Observability, Application Signals, resource discovery hoặc observability pipeline.

Tiếp theo: [rà soát chi phí, bảo mật và dọn dẹp](../5.10-Cost-Security-Cleanup/).
