---
title : "Application Load Balancer"
date: 2026-07-30
weight : 4
chapter : false
pre : " <b> 5.5.4. </b> "
---

Application Load Balancer (ALB) là tài nguyên duy nhất nhận lưu lượng kết nối từ internet. ALB tiếp nhận HTTP cổng 80 và chuyển tiếp đến các máy chủ EC2 tại cổng 3000.

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

| Cấu hình | Giá trị | Giải thích |
|---------|-------|-----|
| `internal` | `false` | Hướng ra Internet — điểm tiếp nhận công khai duy nhất |
| `subnets` | `public_1`, `public_2` | Mỗi AZ một subnet để đảm bảo tính sẵn sàng cao |
| `drop_invalid_header_fields` | `true` | Loại bỏ các request có header HTTP không hợp lệ |
| `target_type` | `instance` | Đăng ký trực tiếp các máy chủ EC2 |
| `health_check.path` | `/health` | Endpoint kiểm tra sức khỏe nhẹ của ứng dụng |
| `healthy_threshold` | 2 | 2 lần kiểm tra thành công liên tiếp trước khi đánh giá healthy |
| `unhealthy_threshold` | 3 | 3 lần kiểm tra thất bại liên tiếp trước khi đánh giá unhealthy |
| `interval` | 30 | Thực hiện kiểm tra mỗi 30 giây |
| `timeout` | 5 | Đánh giá thất bại nếu không nhận phản hồi trong 5 giây |

Target group tự động đăng ký các máy chủ EC2 — liên kết thông qua cấu hình `target_group_arns` của ASG. Khi một máy chủ mới được khởi tạo và vượt qua bài kiểm tra health check, ALB bắt đầu điều hướng lưu lượng đến máy chủ đó. Khi máy chủ lỗi, ALB sẽ dừng chuyển tiếp lưu lượng.

Sau khi chạy `terraform apply`, tên miền ALB DNS sẵn sàng sử dụng:

```bash
terraform output alb_direct_http_url
```
