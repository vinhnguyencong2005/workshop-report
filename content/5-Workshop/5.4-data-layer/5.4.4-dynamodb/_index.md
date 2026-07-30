---
title : "DynamoDB & VPC Endpoint"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4.4. </b> "
---

Five tables using single-table design — each stores multiple entity types in one table with composite keys (PK + SK). All use `PAY_PER_REQUEST` billing and have Point-in-Time Recovery enabled.

#### DynamoDB VPC Gateway Endpoint

Same pattern as S3 — a Gateway endpoint so DynamoDB traffic from private subnets stays on the AWS backbone:

```hcl
data "aws_prefix_list" "dynamodb" {
  name = "com.amazonaws.${var.region}.dynamodb"
}

resource "aws_vpc_endpoint" "dynamodb" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${var.region}.dynamodb"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = [aws_route_table.private_az1.id, aws_route_table.private_az2.id]
  tags              = { Name = "dynamodb-endpoint" }
}

resource "aws_security_group_rule" "ec2_egress_dynamodb" {
  type              = "egress"
  security_group_id = aws_security_group.ec2_sg.id
  from_port         = 443
  to_port           = 443
  protocol          = "tcp"
  prefix_list_ids   = [data.aws_prefix_list.dynamodb.id]
  description       = "HTTPS to DynamoDB via VPC endpoint"
}
```

#### Table Overview

| Table | Purpose | GSIs |
|-------|---------|------|
| `ClassContent` | Course materials, chapters, lessons | `ChapterLookupIndex`, `EntityTypeIndex` |
| `ForumData` | Discussion threads, replies | `ForumConversationsIndex`, `UserActivityIndex`, `EntityTypeIndex` |
| `QuizContent` | Questions, answers, quiz metadata | `QuestionLookupIndex`, `EntityTypeIndex` |
| `StudentSchedule` | Timetables, enrollments | `StudentScheduleIndex`, `EntityTypeIndex` |
| `CourseAssign` | Teacher assignments, wishlists | `CourseWishlistIndex`, `EntityTypeIndex` |

#### Shared Pattern

Every table follows the same structure — PK/SK for primary access, GSIs for alternate query patterns, PITR for point-in-time restore:

```hcl
resource "aws_dynamodb_table" "ClassContent" {
  name         = "ClassContent"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "PK"
  range_key    = "SK"

  attribute { name = "PK"         type = "S" }
  attribute { name = "SK"         type = "S" }
  attribute { name = "GSI1PK"     type = "S" }
  attribute { name = "GSI1SK"     type = "S" }
  attribute { name = "entity_type" type = "S" }

  global_secondary_index {
    name            = "ChapterLookupIndex"
    hash_key        = "GSI1PK"
    range_key       = "GSI1SK"
    projection_type = "ALL"
  }

  global_secondary_index {
    name            = "EntityTypeIndex"
    hash_key        = "entity_type"
    projection_type = "ALL"
  }

  point_in_time_recovery { enabled = true }

  tags = { Name = "ClassContent", Environment = "production" }
}
```

The `EntityTypeIndex` GSI on every table lets us query by entity type across the entire table — useful for admin dashboards and data exports. The table-specific GSIs (e.g., `ChapterLookupIndex`) serve the application's primary query patterns.

`PAY_PER_REQUEST` means no capacity planning — DynamoDB scales automatically and we pay per read/write. PITR allows restoring to any point in the last 35 days.

`CourseAssign` additionally has a TTL attribute:

```hcl
ttl {
  attribute_name = "TimeToExist"
  enabled        = true
}
```

This auto-expires course wishlist entries — students' course preferences are removed after a set period without manual cleanup.