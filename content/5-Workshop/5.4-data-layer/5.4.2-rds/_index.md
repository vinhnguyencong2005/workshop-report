---
title : "RDS MySQL"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
---

A single MySQL 8.0 instance with Multi-AZ for automatic failover. Sits in the DB subnet group spanning `private_3` and `private_4`.

```hcl
resource "aws_db_subnet_group" "main" {
  name       = "main-rds-subnet-group"
  subnet_ids = [aws_subnet.private_3.id, aws_subnet.private_4.id]

  tags = {
    Name = "main-rds-subnet-group"
  }
}

resource "aws_db_instance" "main" {
  identifier     = "main-mysql"
  engine         = "mysql"
  engine_version = var.rds_engine_version
  instance_class = var.rds_instance_class
  db_name        = var.rds_db_name
  username       = var.rds_username
  password       = var.rds_password

  port                   = 3306
  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.rds_sg.id]

  multi_az               = true
  allocated_storage      = 20
  storage_type           = "gp3"
  storage_encrypted      = true
  publicly_accessible    = false
  skip_final_snapshot    = true
  backup_retention_period = 7

  tags = {
    Name = "main-mysql"
  }
}
```

| Setting | Value | Why |
|---------|-------|-----|
| `engine_version` | 8.0 | Latest MySQL major version on RDS |
| `instance_class` | `db.t4g.micro` | Free-tier eligible, ARM-based Graviton2 |
| `multi_az` | `true` | Standby replica in AZ-2; automatic failover if AZ-1 fails |
| `storage_type` | `gp3` | Better price/performance than gp2, baseline 3000 IOPS |
| `storage_encrypted` | `true` | AES-256 encryption at rest using AWS KMS |
| `publicly_accessible` | `false` | No public IP — only reachable from within the VPC |
| `backup_retention_period` | 7 | Automated daily snapshots kept for 7 days |
| `skip_final_snapshot` | `true` | `terraform destroy` won't hang waiting for a final snapshot |
