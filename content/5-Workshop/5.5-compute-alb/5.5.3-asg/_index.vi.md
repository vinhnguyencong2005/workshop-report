---
title : "Auto Scaling Group"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.5.3. </b> "
---

Auto Scaling Group (ASG) duy trì số lượng máy chủ mong muốn hoạt động trên hai private app subnets. Nếu một máy chủ bị lỗi health check hoặc một AZ gặp sự cố, ASG sẽ tự động thay thế máy chủ đó.

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

| Cấu hình | Giá trị | Giải thích |
|---------|-------|-----|
| `vpc_zone_identifier` | `private_1`, `private_2` | Phân bổ máy chủ trên hai AZs — sự cố tại một AZ không làm sập ứng dụng |
| `min_size` | 2 | Một máy chủ cho mỗi AZ — mức tối thiểu cho tính sẵn sàng cao (HA) |
| `desired_capacity` | 2 | Vận hành bình thường 2 máy chủ, tự động tăng khi tải cao |
| `max_size` | 4 | Giới hạn tối đa — ngăn chặn việc tăng máy chủ quá đà khi gặp lỗi |
| `health_check_type` | `ELB` | Sử dụng kiểm tra sức khỏe của ALB target group — phát hiện lỗi ở cấp ứng dụng |
| `health_check_grace_period` | 300 | 5 phút chờ cho script user data hoàn tất bootstrap trước khi kiểm tra sức khỏe |

Cấu hình `health_check_type = "ELB"` rất quan trọng: kiểm tra ở cấp EC2 chỉ phát hiện lỗi phần cứng. Kiểm tra ELB trực tiếp gọi đến endpoint `/health` — nếu ứng dụng Node.js bị sập nhưng máy chủ vẫn chạy, ASG vẫn thực hiện thay thế máy chủ lỗi.

Cấu hình `create_before_destroy` giúp Terraform tạo ASG mới trước khi xóa bản cũ khi cập nhật — đảm bảo không gây gián đoạn dịch vụ.
