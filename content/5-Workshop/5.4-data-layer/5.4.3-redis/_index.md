---
title : "ElastiCache Redis"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.4.3. </b> "
---

Two-node Redis 7.0 replication group — a primary and a replica, one per AZ. Used for session storage and rate limiting.

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

| Setting | Value | Why |
|---------|-------|-----|
| `num_cache_clusters` | 2 | Primary + replica across two AZs |
| `automatic_failover_enabled` | `true` | If primary fails, replica is promoted automatically |
| `multi_az_enabled` | `true` | Replica placed in a different AZ from primary |
| `at_rest_encryption_enabled` | `true` | Data encrypted on disk |
| `transit_encryption_enabled` | `true` | TLS for all client connections |