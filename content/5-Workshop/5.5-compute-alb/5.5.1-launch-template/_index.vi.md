---
title : "Launch Template"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.5.1. </b> "
---

Launch Template định nghĩa cấu hình tiêu chuẩn cho mọi máy chủ EC2 — bao gồm AMI, instance type, security group, IAM profile, đĩa lưu trữ và script bootstrap.

```hcl
resource "aws_launch_template" "app" {
  name_prefix   = "app-lt-"
  image_id      = var.ec2_ami
  instance_type = var.ec2_instance_type

  iam_instance_profile {
    name = aws_iam_instance_profile.ec2.name
  }

  vpc_security_group_ids = [aws_security_group.ec2_sg.id]

  metadata_options {
    http_endpoint               = "enabled"
    http_tokens                 = "required"
    http_put_response_hop_limit = 1
  }

  user_data = base64encode(templatefile("${path.module}/user_data.tftpl", {
    db_host             = split(":", aws_db_instance.main.endpoint)[0]
    db_user             = var.rds_username
    db_password         = var.rds_password
    db_name             = var.rds_db_name
    redis_host          = aws_elasticache_replication_group.main.primary_endpoint_address
    cors_origin         = "*"
    aws_region          = var.region
    uploads_bucket_name = aws_s3_bucket.uploads.bucket
    jwt_secret          = random_password.jwt_secret.result
  }))

  block_device_mappings {
    device_name = "/dev/sda1"
    ebs {
      volume_size           = 30
      volume_type           = "gp3"
      delete_on_termination = true
      encrypted             = true
    }
  }

  tag_specifications {
    resource_type = "instance"
    tags = {
      Name = "app-server-asg"
    }
  }

  lifecycle {
    create_before_destroy = true
  }
}
```

| Cấu hình | Giá trị | Giải thích |
|---------|-------|-----|
| `image_id` | Ubuntu 22.04 (`ami-0c7217cdde317cfec`) | Hệ điều hành ổn định, hỗ trợ các gói phần mềm phổ biến |
| `instance_type` | `t3.medium` | 2 vCPU, 4 GB RAM — đủ cho Node.js + PM2 |
| `http_tokens` | `required` | Bắt buộc IMDSv2 — ngăn chặn các cuộc tấn công SSRF nhắm vào metadata service |
| `volume_size` | 30 GB | Dung lượng cho OS, Node.js, gói npm và logs ứng dụng |
| `volume_type` | `gp3` | Hiệu năng gp3 cao hơn gp2 với cùng mức chi phí |
| `encrypted` | `true` | Mã hóa EBS at-rest |
| `delete_on_termination` | `true` | ASG tự động thay thế máy chủ — không cần giữ ổ đĩa cũ |
| `create_before_destroy` | `true` | Tạo launch template mới trước khi xóa bản cũ khi cập nhật |

#### User Data

Template user data (`user_data.tftpl`) là script bash được Terraform tự động điền các giá trị runtime. Khi máy chủ EC2 khởi động lần đầu:

1. **Cài đặt các gói hệ thống** — git, mysql-client, AWS CLI v2, Node.js 20 qua nvm, PM2
2. **Pull mã nguồn backend** — tải `backend-latest.zip` từ S3 deployments bucket
3. **Tạo tệp `.env`** — nạp DB host, Redis host, S3 bucket name, JWT secret từ outputs của Terraform
4. **Cài đặt npm dependencies** và khởi chạy ứng dụng với PM2

```bash
# Trích đoạn chính của user_data.tftpl:

# Tải code từ S3
aws s3 cp s3://app-backend-deployments-agy/backend-latest.zip /tmp/backend.zip

# Tạo .env với các giá trị được nạp từ Terraform
cat > /home/ubuntu/app/.env <<EOF
PORT=3000
DB_HOST=${db_host}
DB_USER=${db_user}
DB_PASSWORD=${db_password}
DB_NAME=${db_name}
REDIS_HOST=${redis_host}
REDIS_URL=rediss://${redis_host}:6379
AWS_REGION=${aws_region}
S3_BUCKET_NAME=${uploads_bucket_name}
JWT_SECRET=${jwt_secret}
EOF

# Khởi chạy ứng dụng
npm ci --production
pm2 start src/index.js --name "app"
pm2 save
```
