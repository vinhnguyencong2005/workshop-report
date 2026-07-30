---
title : "Application Load Balancer"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.5.4. </b> "
---

The ALB is the only resource with a public IP. It accepts HTTP on port 80 and forwards to EC2 instances on port 3000.

```hcl
resource "aws_lb" "main" {
  name                       = "main-alb"
  internal                   = false
  load_balancer_type         = "application"
  security_groups            = [aws_security_group.alb_sg.id]
  subnets                    = [aws_subnet.public_1.id, aws_subnet.public_2.id]
  drop_invalid_header_fields = true

  tags = { Name = "main-alb" }
}

resource "aws_lb_target_group" "app" {
  name        = "app-tg"
  port        = 3000
  protocol    = "HTTP"
  vpc_id      = aws_vpc.main.id
  target_type = "instance"

  health_check {
    path                = "/health"
    protocol            = "HTTP"
    port                = "3000"
    healthy_threshold   = 2
    unhealthy_threshold = 3
    interval            = 30
    timeout             = 5
    matcher             = "200-299"
  }

  tags = { Name = "app-tg" }
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}
```

| Setting | Value | Why |
|---------|-------|-----|
| `internal` | `false` | Internet-facing — the only public entry point |
| `subnets` | `public_1`, `public_2` | One per AZ for HA |
| `drop_invalid_header_fields` | `true` | Drops requests with malformed HTTP headers |
| `target_type` | `instance` | Registers EC2 instances directly (not IP) |
| `health_check.path` | `/health` | App exposes a lightweight health endpoint |
| `healthy_threshold` | 2 | Two consecutive successes before marking healthy |
| `unhealthy_threshold` | 3 | Three consecutive failures before marking unhealthy |
| `interval` | 30 | Check every 30 seconds |
| `timeout` | 5 | Mark as failed if no response in 5 seconds |

The target group registers EC2 instances automatically — the ASG's `target_group_arns` links them. When a new instance launches and passes health checks, the ALB starts routing traffic to it. When an instance fails, the ALB stops.

Currently HTTP-only (port 80). HTTPS with ACM certificate can be added later.

After `terraform apply`, the ALB DNS name is available as:

```bash
terraform output backend_api_url
# http://main-alb-123456789.us-east-1.elb.amazonaws.com
```

---

✅ **Next:** [Monitoring](5.6-monitoring/) — CloudWatch alarms for ALB errors, RDS CPU, and ASG health.
