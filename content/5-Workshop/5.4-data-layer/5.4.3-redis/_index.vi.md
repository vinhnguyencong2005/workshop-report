---
title : "ElastiCache Redis"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.4.3. </b> "
---

Replication group Redis 7.0 gồm hai nodes — một primary và một replica, đặt trên từng AZ. Được sử dụng cho quản lý phiên đăng nhập (session storage) và giới hạn tần suất truy cập (rate limiting).

```hcl
resource "aws_elasticache_subnet_group" "main" {
  name       = "main-redis-subnet-group"
  subnet_ids = [aws_subnet.private_3.id, aws_subnet.private_4.id]

  tags = {
    Name = "main-redis-subnet-group"
  }
}

resource "aws_elasticache_replication_group" "main" {
  replication_group_id = "main-redis"
  description          = "Redis replication group primary and replica"

  engine               = "redis"
  engine_version       = var.redis_engine_version
  node_type            = var.redis_node_type
  parameter_group_name = "default.redis7"
  port                 = 6379

  num_cache_clusters         = 2
  automatic_failover_enabled = true
  multi_az_enabled           = true

  at_rest_encryption_enabled  = true
  transit_encryption_enabled  = true

  subnet_group_name  = aws_elasticache_subnet_group.main.name
  security_group_ids = [aws_security_group.redis_sg.id]

  tags = {
    Name = "main-redis"
  }
}
```

| Cấu hình | Giá trị | Giải thích |
|---------|-------|-----|
| `num_cache_clusters` | 2 | Primary + replica trải rộng trên hai AZs |
| `automatic_failover_enabled` | `true` | Nếu node primary lỗi, node replica tự động được nâng cấp |
| `multi_az_enabled` | `true` | Node replica đặt tại AZ khác với node primary |
| `at_rest_encryption_enabled` | `true` | Mã hóa dữ liệu trên đĩa đĩa |
| `transit_encryption_enabled` | `true` | Bật TLS cho toàn bộ kết nối từ client |
