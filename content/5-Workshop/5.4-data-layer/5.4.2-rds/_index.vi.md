---
title : "RDS MySQL"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
---

Một instance MySQL 8.0 với Multi-AZ hỗ trợ tự động khôi phục khi gặp sự cố (failover). Đặt trong DB subnet group trải dài trên `private_3` và `private_4`.

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

| Cấu hình | Giá trị | Giải thích |
|---------|-------|-----|
| `engine_version` | 8.0 | Phiên bản MySQL mới nhất trên RDS |
| `instance_class` | `db.t4g.micro` | Tiết kiệm chi phí, chip ARM Graviton2 |
| `multi_az` | `true` | Bản sao dự phòng tại AZ-2; tự động chuyển đổi khi AZ-1 lỗi |
| `storage_type` | `gp3` | Hiệu năng tốt hơn gp2, mặc định 3000 IOPS |
| `storage_encrypted` | `true` | Mã hóa AES-256 dữ liệu lưu trữ bằng AWS KMS |
| `publicly_accessible` | `false` | Không cấp IP công khai — chỉ truy cập nội bộ từ VPC |
| `backup_retention_period` | 7 | Tự động chụp bản sao lưu (snapshot) hàng ngày và lưu trong 7 ngày |
| `skip_final_snapshot` | `true` | Lệnh `terraform destroy` xóa ngay mà không chờ chụp snapshot cuối cùng |
