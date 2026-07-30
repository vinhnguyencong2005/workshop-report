---
title : "Security Groups"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.3.4. </b> "
---

Năm security groups, mỗi nhóm đảm nhận một nhiệm vụ riêng biệt:

```
Internet ──→ [alb-sg] ──→ [ec2-sg] ──→ [rds-sg]
                           │    │
                           │    └──→ [redis-sg]
                           │
                    [vpce-sg] (SSM endpoints)
```

#### 1. ALB (`alb-sg`)

Security Group duy nhất chấp nhận lưu lượng truy cập từ Internet:

```hcl
resource "aws_security_group" "alb_sg" {
  name        = "alb-sg"
  description = "Allow HTTP inbound from internet; forward to EC2 app port 3000"
  vpc_id      = aws_vpc.main.id
  tags        = { Name = "alb-sg" }
}

resource "aws_security_group_rule" "alb_ingress_http" {
  type              = "ingress"
  security_group_id = aws_security_group.alb_sg.id
  from_port         = 80
  to_port           = 80
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]
  description       = "HTTP from internet"
}

resource "aws_security_group_rule" "alb_egress_ec2" {
  type                     = "egress"
  security_group_id        = aws_security_group.alb_sg.id
  from_port                = 3000
  to_port                  = 3000
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ec2_sg.id
  description              = "Forward to EC2 app port 3000"
}
```

| Hướng | Cổng (Port) | Nguồn/Đích | Mục đích |
|-----------|------|-------------|--------|
| Ingress | 80 | `0.0.0.0/0` | Chấp nhận HTTP từ Internet |
| Egress | 3000 | `ec2-sg` | Chuyển tiếp request đến EC2 |

#### 2. EC2 (`ec2-sg`)

Chỉ chấp nhận lưu lượng từ ALB; kết nối ra cơ sở dữ liệu và các dịch vụ AWS:

```hcl
resource "aws_security_group" "ec2_sg" {
  name        = "ec2-sg"
  description = "EC2 app-tier security group"
  vpc_id      = aws_vpc.main.id
  tags        = { Name = "ec2-sg" }
}

# Ingress: Chỉ ALB được phép gửi lưu lượng đến ứng dụng
resource "aws_security_group_rule" "ec2_ingress_alb" {
  type                     = "ingress"
  security_group_id        = aws_security_group.ec2_sg.id
  from_port                = 3000
  to_port                  = 3000
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.alb_sg.id
  description              = "App traffic from ALB only"
}

# Egress: Truy cập RDS MySQL
resource "aws_security_group_rule" "ec2_egress_rds" {
  type                     = "egress"
  security_group_id        = aws_security_group.ec2_sg.id
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.rds_sg.id
  description              = "MySQL to RDS"
}

# Egress: Truy cập ElastiCache Redis
resource "aws_security_group_rule" "ec2_egress_redis" {
  type                     = "egress"
  security_group_id        = aws_security_group.ec2_sg.id
  from_port                = 6379
  to_port                  = 6379
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.redis_sg.id
  description              = "Redis to ElastiCache"
}

# Egress: Tất cả outbound đi qua NAT Gateway (apt, npm, HTTPS)
resource "aws_security_group_rule" "ec2_egress_all_outbound" {
  type              = "egress"
  security_group_id = aws_security_group.ec2_sg.id
  from_port         = 0
  to_port           = 0
  protocol          = "-1"
  cidr_blocks       = ["0.0.0.0/0"]
  description       = "All outbound traffic via NAT Gateway"
}

# Egress: Phân giải DNS
resource "aws_security_group_rule" "ec2_egress_dns_tcp" {
  type              = "egress"
  security_group_id = aws_security_group.ec2_sg.id
  from_port         = 53
  to_port           = 53
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]
  description       = "DNS TCP"
}

resource "aws_security_group_rule" "ec2_egress_dns_udp" {
  type              = "egress"
  security_group_id = aws_security_group.ec2_sg.id
  from_port         = 53
  to_port           = 53
  protocol          = "udp"
  cidr_blocks       = ["0.0.0.0/0"]
  description       = "DNS UDP"
}
```

| Hướng | Cổng (Port) | Nguồn/Đích | Mục đích |
|-----------|------|-------------|--------|
| Ingress | 3000 | `alb-sg` | Chỉ ALB kết nối đến ứng dụng |
| Egress | 3306 | `rds-sg` | Truy vấn MySQL đến RDS |
| Egress | 6379 | `redis-sg` | Đọc/Ghi cache đến Redis |
| Egress | 53 (TCP+UDP) | `0.0.0.0/0` | Phân giải DNS |
| Egress | Tất cả | `0.0.0.0/0` | Outbound qua NAT (apt, npm, HTTPS) |

Các luật egress cho S3 và DynamoDB (cổng 443 qua prefix lists) được thêm cùng các VPC Endpoints tại tầng dữ liệu.

#### 3. RDS (`rds-sg`)

```hcl
resource "aws_security_group" "rds_sg" {
  name        = "rds-sg"
  description = "Allow MySQL from EC2"
  vpc_id      = aws_vpc.main.id
  tags        = { Name = "rds-sg" }
}

resource "aws_security_group_rule" "rds_ingress_mysql" {
  type                     = "ingress"
  security_group_id        = aws_security_group.rds_sg.id
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ec2_sg.id
  description              = "MySQL from EC2 only"
}
```

| Hướng | Cổng (Port) | Nguồn | Mục đích |
|-----------|------|--------|--------|
| Ingress | 3306 | `ec2-sg` | Chỉ các máy chủ EC2 được truy vấn MySQL |

#### 4. Redis (`redis-sg`)

```hcl
resource "aws_security_group" "redis_sg" {
  name        = "redis-sg"
  description = "Allow Redis from EC2"
  vpc_id      = aws_vpc.main.id
  tags        = { Name = "redis-sg" }
}

resource "aws_security_group_rule" "redis_ingress" {
  type                     = "ingress"
  security_group_id        = aws_security_group.redis_sg.id
  from_port                = 6379
  to_port                  = 6379
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ec2_sg.id
  description              = "Redis from EC2 only"
}
```

| Hướng | Cổng (Port) | Nguồn | Mục đích |
|-----------|------|--------|--------|
| Ingress | 6379 | `ec2-sg` | Chỉ các máy chủ EC2 được kết nối Redis |

#### 5. VPC Endpoints (`vpce-sg`)

Sử dụng cho SSM Interface Endpoints để cho phép HTTPS nội bộ từ VPC:

```hcl
resource "aws_security_group" "vpce_sg" {
  name        = "vpce-sg"
  description = "Allow HTTPS inbound from VPC for VPC Interface Endpoints"
  vpc_id      = aws_vpc.main.id
  tags        = { Name = "vpce-sg" }
}

resource "aws_security_group_rule" "vpce_ingress_https" {
  type              = "ingress"
  security_group_id = aws_security_group.vpce_sg.id
  from_port         = 443
  to_port           = 443
  protocol          = "tcp"
  cidr_blocks       = [var.vpc_cidr]
  description       = "HTTPS from VPC"
}
```

Hầu hết các luật đều tham chiếu đến các security groups khác (thay vì IP cố định), giúp cấu hình tự động thích ứng khi số lượng máy chủ tăng giảm.
