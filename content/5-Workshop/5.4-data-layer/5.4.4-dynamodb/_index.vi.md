---
title : "DynamoDB & VPC Endpoint"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4.4. </b> "
---

Năm bảng sử dụng thiết kế single-table — mỗi bảng lưu trữ nhiều loại thực thể với khóa tổng hợp (PK + SK). Tất cả đều sử dụng chế độ thanh toán `PAY_PER_REQUEST` và bật tính năng Point-in-Time Recovery (PITR).

#### DynamoDB VPC Gateway Endpoint

Tương tự như S3 — một Gateway endpoint giúp lưu lượng DynamoDB từ các private subnets đi trên hạ tầng AWS backbone:

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

#### Tổng quan các Bảng (Tables)

| Bảng (Table) | Mục đích sử dụng | GSIs |
|-------|---------|------|
| `ClassContent` | Tài liệu khóa học, chương, bài học | `ChapterLookupIndex`, `EntityTypeIndex` |
| `ForumData` | Chuỗi thảo luận, câu trả lời | `ForumConversationsIndex`, `UserActivityIndex`, `EntityTypeIndex` |
| `QuizContent` | Câu hỏi, đáp án, dữ liệu bài kiểm tra | `QuestionLookupIndex`, `EntityTypeIndex` |
| `StudentSchedule` | Thời khóa biểu, danh sách đăng ký | `StudentScheduleIndex`, `EntityTypeIndex` |
| `CourseAssign` | Phân công giảng viên, danh sách nguyện vọng | `CourseWishlistIndex`, `EntityTypeIndex` |

#### Mẫu Cấu hình Chung (Shared Pattern)

Mỗi bảng tuân theo một cấu trúc thống nhất — PK/SK cho truy xuất chính, GSIs cho các mẫu truy vấn thay thế, PITR cho việc khôi phục dữ liệu theo mốc thời gian:

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

GSI `EntityTypeIndex` trên mỗi bảng cho phép truy vấn theo loại thực thể trên toàn bộ bảng — hữu ích cho dashboard quản trị và xuất dữ liệu. Các GSI riêng (ví dụ `ChapterLookupIndex`) phục vụ các mẫu truy vấn chính của ứng dụng.

Chế độ `PAY_PER_REQUEST` giúp DynamoDB tự động co giãn và chỉ tính phí theo lượng Read/Write thực tế. PITR hỗ trợ khôi phục dữ liệu về bất kỳ thời điểm nào trong 35 ngày gần nhất.

Bảng `CourseAssign` bổ sung thêm thuộc tính TTL:

```hcl
ttl {
  attribute_name = "TimeToExist"
  enabled        = true
}
```

Thuộc tính này giúp tự động hết hạn các nguyện vọng khóa học sau một khoảng thời gian thiết lập mà không cần dọn dẹp thủ công.
