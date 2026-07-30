---
title : "Auto Scaling Group"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.5.3. </b> "
---

The ASG maintains the desired number of healthy instances across two private app subnets. If an instance fails health checks or an AZ goes down, the ASG replaces it automatically.

```hcl
resource "aws_autoscaling_group" "app" {
  name_prefix         = "app-asg-"
  vpc_zone_identifier = [aws_subnet.private_1.id, aws_subnet.private_2.id]

  min_size         = var.asg_min_size
  desired_capacity = var.asg_desired_capacity
  max_size         = var.asg_max_size

  target_group_arns         = [aws_lb_target_group.app.arn]
  health_check_type         = "ELB"
  health_check_grace_period = 300

  launch_template {
    id      = aws_launch_template.app.id
    version = "$Latest"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag {
    key                 = "Name"
    value               = "app-server-asg"
    propagate_at_launch = true
  }
}
```

| Setting | Value | Why |
|---------|-------|-----|
| `vpc_zone_identifier` | `private_1`, `private_2` | Instances spread across two AZs — no single-AZ failure takes out the app |
| `min_size` | 2 | One instance per AZ — minimum for HA |
| `desired_capacity` | 2 | Runs 2 instances normally, scales up under load |
| `max_size` | 4 | Ceiling — prevents runaway scaling if a bug triggers constant scale-out |
| `health_check_type` | `ELB` | Uses ALB target group health checks — catches app-level failures, not just EC2 status |
| `health_check_grace_period` | 300 | 5 minutes for user data to finish bootstrapping before health checks start |

`health_check_type = "ELB"`: EC2-level health checks only detect hypervisor failures. ELB health checks hit the `/health` endpoint — if the Node.js app crashes but the instance is running, the ASG still replaces it.

`create_before_destroy` on the lifecycle means Terraform creates the new ASG before destroying the old one during updates — no downtime during config changes.
