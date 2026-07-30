---
title : "Giám sát & Cảnh báo (Monitoring)"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

Một SNS topic kết hợp với ba CloudWatch alarms bao phủ từng tầng hạ tầng có nguy cơ gặp sự cố. Khi một alarm kích hoạt, thông báo sẽ được xuất bản đến SNS topic — hỗ trợ đăng ký email hoặc webhook để nhận thông báo thời gian thực.

#### SNS Topic

```hcl
resource "aws_sns_topic" "alarms" {
  name = "infra-alarms"
  tags = { Name = "infra-alarms" }
}
```

#### Các Cảnh báo (Alarms)

| Cảnh báo (Alarm) | Chỉ số (Metric) | Ngưỡng (Threshold) | Logic kiểm tra |
|-------|--------|-----------|-------|
| `alb-5xx-errors` | `HTTPCode_Target_5XX_Count` | > 5 trong 2 chu kỳ 1 phút | Ứng dụng trả về lỗi 5xx cho người dùng |
| `rds-cpu-high` | `CPUUtilization` | > 80% trong 2 chu kỳ 5 phút | Cơ sở dữ liệu bị quá tải — cần tối ưu truy vấn hoặc nâng cấp máy chủ |
| `asg-below-min-size` | `GroupTotalInstances` | < 2 trong 2 chu kỳ 5 phút | Máy chủ EC2 khởi động thất bại hoặc bị dừng bất thường |

#### 1. Cảnh báo Lỗi ALB 5xx

```hcl
resource "aws_cloudwatch_metric_alarm" "alb_5xx" {
  alarm_name          = "alb-5xx-errors"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  threshold           = 5
  period              = 60
  namespace           = "AWS/ApplicationELB"
  metric_name         = "HTTPCode_Target_5XX_Count"
  statistic           = "Sum"
  treat_missing_data  = "notBreaching"

  dimensions = {
    LoadBalancer = aws_lb.main.arn_suffix
  }

  alarm_actions = [aws_sns_topic.alarms.arn]
  ok_actions    = [aws_sns_topic.alarms.arn]
}
```

Cấu hình `treat_missing_data = "notBreaching"` — nếu ALB không có lưu lượng (và do đó không phát sinh dữ liệu 5xx), trạng thái được coi là bình thường. Cảnh báo chỉ kích hoạt khi có dữ liệu thực tế vượt ngưỡng.

#### 2. Cảnh báo Quá tải CPU RDS

```hcl
resource "aws_cloudwatch_metric_alarm" "rds_cpu" {
  alarm_name          = "rds-cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  threshold           = 80
  period              = 300
  namespace           = "AWS/RDS"
  metric_name         = "CPUUtilization"
  statistic           = "Average"
  treat_missing_data  = "notBreaching"

  dimensions = {
    DBInstanceIdentifier = aws_db_instance.main.identifier
  }

  alarm_actions = [aws_sns_topic.alarms.arn]
  ok_actions    = [aws_sns_topic.alarms.arn]
}
```

CPU duy trì ở mức 80% liên tục trong 10 phút (2 chu kỳ 5 phút). Với cấu hình `db.t4g.micro`, điều này cho thấy cơ sở dữ liệu đang đạt tới giới hạn CPU. Phương án xử lý: nâng cấp cấu hình máy chủ hoặc thêm read replica.

#### 3. Cảnh báo ASG Dưới Mức Tối thiểu

```hcl
resource "aws_cloudwatch_metric_alarm" "asg_below_min" {
  alarm_name          = "asg-below-min-size"
  comparison_operator = "LessThanThreshold"
  evaluation_periods  = 2
  threshold           = var.asg_min_size
  period              = 300
  namespace           = "AWS/AutoScaling"
  metric_name         = "GroupTotalInstances"
  statistic           = "Average"
  treat_missing_data  = "breaching"

  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.app.name
  }

  alarm_actions = [aws_sns_topic.alarms.arn]
  ok_actions    = [aws_sns_topic.alarms.arn]
}
```

Cấu hình `treat_missing_data = "breaching"` — ngược lại với hai cảnh báo trên. Nếu không thể lấy dữ liệu từ ASG, hệ thống xác định có bất thường xảy ra (ASG có thể đã bị xóa hoặc gặp sự cố).

#### Ý nghĩa của ba cảnh báo chính

- **ALB 5xx** — Phát hiện lỗi mã nguồn ứng dụng, thất bại khi triển khai code, hoặc dịch vụ phụ thuộc bị gián đoạn.
- **RDS CPU** — Phát hiện hiệu năng truy vấn kém, thiếu chỉ mục (index), hoặc lưu lượng truy cập tăng đột biến.
- **ASG below min** — Phát hiện lỗi khởi tạo máy chủ, gián đoạn tại một AZ, hoặc thiếu hụt dung lượng máy chủ.
