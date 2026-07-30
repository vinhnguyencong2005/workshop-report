---
title : "IAM Role cho EC2"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.3.5. </b> "
---

EC2 cần gọi đến S3, DynamoDB, RDS, ElastiCache, và Secrets Manager. Thay vì sử dụng access keys trong mã nguồn, giải pháp áp dụng IAM role để EC2 đảm nhận qua dịch vụ Instance Metadata Service — thông tin xác thực xoay vòng tự động, tránh lộ chìa khóa bảo mật.

Bao gồm bốn thành phần:

#### 1. Trust Policy — Chỉ EC2 mới có thể đảm nhận role này

```hcl
data "aws_iam_policy_document" "ec2_assume_role" {
  statement {
    actions = ["sts:AssumeRole"]
    principals {
      type        = "Service"
      identifiers = ["ec2.amazonaws.com"]
    }
  }
}

resource "aws_iam_role" "ec2_role" {
  name               = "ec2-app-role"
  assume_role_policy = data.aws_iam_policy_document.ec2_assume_role.json
  tags               = { Name = "ec2-app-role" }
}
```

#### 2. Inline Policy — Được giới hạn chính xác theo nhu cầu ứng dụng

| Dịch vụ | Hành động (Actions) | Tài nguyên (Resources) |
|---------|---------|-----------|
| S3 | `PutObject`, `GetObject`, `DeleteObject` | Chỉ Uploads bucket |
| S3 | `GetObject` | Chỉ Deployments bucket |
| DynamoDB | Full CRUD + Query + Scan | 5 bảng + các chỉ mục |
| ElastiCache | `Connect` | Redis replication group |
| RDS | `rds-db:connect` | Tất cả |
| Secrets Manager | `GetSecretValue` | Secret khóa riêng tư JWT |

Mỗi khai báo đều sử dụng ARN tài nguyên hẹp nhất có thể — policy S3 uploads được giới hạn trong `"${aws_s3_bucket.uploads.arn}/*"`, không sử dụng `"*"`.

```hcl
data "aws_iam_policy_document" "ec2_policy" {
  statement {
    sid       = "S3Uploads"
    effect    = "Allow"
    actions   = ["s3:PutObject", "s3:GetObject", "s3:DeleteObject"]
    resources = ["${aws_s3_bucket.uploads.arn}/*"]
  }

  statement {
    sid       = "S3Deployments"
    effect    = "Allow"
    actions   = ["s3:GetObject"]
    resources = ["${aws_s3_bucket.deployments.arn}/*"]
  }

  statement {
    sid    = "DynamoDB"
    effect = "Allow"
    actions = [
      "dynamodb:GetItem", "dynamodb:PutItem", "dynamodb:UpdateItem",
      "dynamodb:DeleteItem", "dynamodb:Query", "dynamodb:Scan",
      "dynamodb:BatchGetItem", "dynamodb:BatchWriteItem",
      "dynamodb:DescribeTable", "dynamodb:ListTables"
    ]
    resources = [
      aws_dynamodb_table.ClassContent.arn,
      "${aws_dynamodb_table.ClassContent.arn}/index/*",
      aws_dynamodb_table.ForumData.arn,
      "${aws_dynamodb_table.ForumData.arn}/index/*",
      aws_dynamodb_table.QuizContent.arn,
      "${aws_dynamodb_table.QuizContent.arn}/index/*",
      aws_dynamodb_table.StudentSchedule.arn,
      "${aws_dynamodb_table.StudentSchedule.arn}/index/*",
      aws_dynamodb_table.CourseAssign.arn,
      "${aws_dynamodb_table.CourseAssign.arn}/index/*"
    ]
  }

  statement {
    sid       = "ElastiCacheConnect"
    effect    = "Allow"
    actions   = ["elasticache:Connect"]
    resources = [aws_elasticache_replication_group.main.arn]
  }

  statement {
    sid       = "RDSConnect"
    effect    = "Allow"
    actions   = ["rds-db:connect"]
    resources = ["*"]
  }

  statement {
    sid       = "SecretsManager"
    effect    = "Allow"
    actions   = ["secretsmanager:GetSecretValue"]
    resources = [var.jwt_private_key_secret_arn]
  }
}

resource "aws_iam_role_policy" "ec2_policy" {
  name   = "ec2-app-policy"
  role   = aws_iam_role.ec2_role.id
  policy = data.aws_iam_policy_document.ec2_policy.json
}
```

#### 3. SSM Managed Policy — Kích hoạt Session Manager

```hcl
resource "aws_iam_role_policy_attachment" "ec2_ssm" {
  role       = aws_iam_role.ec2_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
}
```

Nếu không có policy này, lệnh `aws ssm start-session` sẽ báo lỗi. Đây là cơ chế giúp tránh hoàn toàn việc sử dụng SSH và bastion hosts.

#### 4. Instance Profile — Đóng gói role để EC2 có thể sử dụng

```hcl
resource "aws_iam_instance_profile" "ec2" {
  name = "ec2-app-profile"
  role = aws_iam_role.ec2_role.name
}
```
