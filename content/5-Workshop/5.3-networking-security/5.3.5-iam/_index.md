---
title : "IAM Role for EC2"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.3.5. </b> "
---

EC2 needs to call S3, DynamoDB, RDS, ElastiCache, and Secrets Manager. Instead of access keys in code, we use an IAM role that EC2 assumes via the instance metadata service — credentials rotate automatically, nothing to leak.

Four pieces:

#### 1. Trust Policy — only EC2 can assume this role

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

#### 2. Inline Policy — scoped to exactly what the app needs

| Service | Actions | Resources |
|---------|---------|-----------|
| S3 | `PutObject`, `GetObject`, `DeleteObject` | Uploads bucket only |
| S3 | `GetObject` | Deployments bucket only |
| DynamoDB | Full CRUD + Query + Scan | 5 tables + their indexes |
| ElastiCache | `Connect` | Redis replication group |
| RDS | `rds-db:connect` | All |
| Secrets Manager | `GetSecretValue` | JWT private key secret |

Each statement uses the narrowest possible resource ARN — the S3 uploads policy scopes to `"${aws_s3_bucket.uploads.arn}/*"`, not `"*"`.

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

#### 3. SSM Managed Policy — enables Session Manager

```hcl
resource "aws_iam_role_policy_attachment" "ec2_ssm" {
  role       = aws_iam_role.ec2_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
}
```

Without this, `aws ssm start-session` fails. This is how we avoid SSH and bastion hosts.

#### 4. Instance Profile — wraps the role so EC2 can use it

```hcl
resource "aws_iam_instance_profile" "ec2" {
  name = "ec2-app-profile"
  role = aws_iam_role.ec2_role.name
}
```