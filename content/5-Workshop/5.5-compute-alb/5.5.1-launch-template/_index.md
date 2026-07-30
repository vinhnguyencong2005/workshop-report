---
title : "Launch Template"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.5.1. </b> "
---

The launch template defines what every EC2 instance looks like — the AMI, instance type, security group, IAM profile, disk, and bootstrap script.

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

| Setting | Value | Why |
|---------|-------|-----|
| `image_id` | Ubuntu 22.04 (`ami-0c7217cdde317cfec`) | Familiar OS, broad package support |
| `instance_type` | `t3.medium` | 2 vCPU, 4 GB RAM — enough for Node.js + PM2 |
| `http_tokens` | `required` | Enforces IMDSv2 — blocks SSRF attacks that target the metadata service |
| `volume_size` | 30 GB | Room for OS, Node.js, npm packages, and application logs |
| `volume_type` | `gp3` | Better baseline performance than gp2, same cost |
| `encrypted` | `true` | EBS encryption at rest |
| `delete_on_termination` | `true` | ASG replaces instances — no need to keep old volumes |
| `create_before_destroy` | `true` | New launch template is created before old one is deleted during updates |

#### User Data

The user data template (`user_data.tftpl`) is a bash script that Terraform populates with runtime values. On first boot, each EC2 instance:

1. **Installs system packages** — git, mysql-client, AWS CLI v2, Node.js 20 via nvm, PM2
2. **Pulls the backend code** — downloads `backend-latest.zip` from the deployments S3 bucket
3. **Writes the `.env` file** — injects DB host, Redis host, S3 bucket name, JWT secret from Terraform outputs
4. **Installs npm dependencies** and starts the app with PM2

```bash
# Key parts of user_data.tftpl:

# Pull code from S3
aws s3 cp s3://app-backend-deployments-agy/backend-latest.zip /tmp/backend.zip

# Generate .env with Terraform-injected values
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

# Start the app
npm ci --production
pm2 start src/index.js --name "app"
pm2 save
```

All `${...}` values in the template are replaced by Terraform at apply time — no hardcoded secrets in the script.