---
title : "Security Groups"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.3.4. </b> "
---

Five security groups, each with one responsibility:

```
Internet ──→ [alb-sg] ──→ [ec2-sg] ──→ [rds-sg]
                           │    │
                           │    └──→ [redis-sg]
                           │
                    [vpce-sg] (SSM endpoints)
```

#### 1. ALB (`alb-sg`)

The only SG that accepts traffic from the internet:

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

| Direction | Port | Source/Dest | Reason |
|-----------|------|-------------|--------|
| Ingress | 80 | `0.0.0.0/0` | Accept HTTP from the internet |
| Egress | 3000 | `ec2-sg` | Forward requests to EC2 |

#### 2. EC2 (`ec2-sg`)

Accepts traffic only from the ALB; reaches out to databases and AWS services:

```hcl
resource "aws_security_group" "ec2_sg" {
  name        = "ec2-sg"
  description = "EC2 app-tier security group"
  vpc_id      = aws_vpc.main.id
  tags        = { Name = "ec2-sg" }
}

# Ingress: Only the ALB can send traffic to our app
resource "aws_security_group_rule" "ec2_ingress_alb" {
  type                     = "ingress"
  security_group_id        = aws_security_group.ec2_sg.id
  from_port                = 3000
  to_port                  = 3000
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.alb_sg.id
  description              = "App traffic from ALB only"
}

# Egress: Reach RDS MySQL
resource "aws_security_group_rule" "ec2_egress_rds" {
  type                     = "egress"
  security_group_id        = aws_security_group.ec2_sg.id
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.rds_sg.id
  description              = "MySQL to RDS"
}

# Egress: Reach ElastiCache Redis
resource "aws_security_group_rule" "ec2_egress_redis" {
  type                     = "egress"
  security_group_id        = aws_security_group.ec2_sg.id
  from_port                = 6379
  to_port                  = 6379
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.redis_sg.id
  description              = "Redis to ElastiCache"
}

# Egress: All outbound via NAT Gateway (apt, npm, HTTPS)
resource "aws_security_group_rule" "ec2_egress_all_outbound" {
  type              = "egress"
  security_group_id = aws_security_group.ec2_sg.id
  from_port         = 0
  to_port           = 0
  protocol          = "-1"
  cidr_blocks       = ["0.0.0.0/0"]
  description       = "All outbound traffic via NAT Gateway"
}

# Egress: DNS resolution
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

| Direction | Port | Source/Dest | Reason |
|-----------|------|-------------|--------|
| Ingress | 3000 | `alb-sg` | Only the ALB reaches our app |
| Egress | 3306 | `rds-sg` | MySQL queries to RDS |
| Egress | 6379 | `redis-sg` | Cache reads/writes to Redis |
| Egress | 53 (TCP+UDP) | `0.0.0.0/0` | DNS resolution |
| Egress | All | `0.0.0.0/0` | Outbound via NAT (apt, npm, HTTPS) |

S3 and DynamoDB egress rules (port 443 via prefix lists) are added later alongside their VPC Endpoints in the [Data Layer](5.4-data-layer/).

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

| Direction | Port | Source | Reason |
|-----------|------|--------|--------|
| Ingress | 3306 | `ec2-sg` | Only EC2 instances can query MySQL |

No egress rules needed — RDS never initiates outbound connections that we need to control.

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

| Direction | Port | Source | Reason |
|-----------|------|--------|--------|
| Ingress | 6379 | `ec2-sg` | Only EC2 instances can access Redis |

#### 5. VPC Endpoints (`vpce-sg`)

Used by SSM Interface Endpoints to allow HTTPS from within the VPC:

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

| Direction | Port | Source | Reason |
|-----------|------|--------|--------|
| Ingress | 443 | VPC CIDR | Allow Session Manager connections from any resource in the VPC |

Most rules reference other security groups (not IPs), so rules don't need changes when instances scale up or down.