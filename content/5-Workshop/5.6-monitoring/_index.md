---
title : "Monitoring"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

One SNS topic + three CloudWatch alarms covering each tier that can fail. When an alarm fires, it publishes to the SNS topic — subscribe an email or webhook to receive notifications.

#### SNS Topic

```hcl
resource "aws_sns_topic" "alarms" {
  name = "infra-alarms"
  tags = { Name = "infra-alarms" }
}
```

#### Alarms

| Alarm | Metric | Threshold | Logic |
|-------|--------|-----------|-------|
| `alb-5xx-errors` | `HTTPCode_Target_5XX_Count` | > 5 over 2× 1-min periods | App returning errors to users |
| `rds-cpu-high` | `CPUUtilization` | > 80% over 2 × 5-min periods | Assume database under pressure — needs scaling or query optimization |
| `asg-below-min-size` | `GroupTotalInstances` | < 2 over 2 × 5-min periods | Instances failing to launch or being terminated |

#### ALB 5xx

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

`treat_missing_data = "notBreaching"` — if the ALB has no traffic (and therefore no 5xx data points), it's not a problem. Only fire when we have data exceeding the threshold.

#### RDS CPU

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

80% CPU sustained over 10 minutes (2 × 5-min periods). At `db.t4g.micro` with 2 vCPUs, this means the database is CPU-bound. Response: scale up instance class or add a read replica.

#### ASG Below Minimum

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

`treat_missing_data = "breaching"` — opposite of the other two. If we can't get data from the ASG at all, something is wrong. Unlike ALB (no traffic = fine), ASG (no data = the ASG itself might be deleted or broken).

#### Why These Three?

- **ALB 5xx** — catches application bugs, deployment failures, dependency outages
- **RDS CPU** — catches query performance issues, missing indexes, traffic spikes
- **ASG below min** — catches launch failures, AZ outages, instance type capacity issues

Each covers a different failure mode. Together they give enough signal to know when to investigate without generating noise.