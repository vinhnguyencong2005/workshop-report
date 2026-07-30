---
title : "S3 Buckets & VPC Endpoint"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

Three buckets, each with different access patterns:

| Bucket | Security | Key Settings |
|--------|---------|-------------|
| Frontend | Public read | Website hosting, bucket policy for `s3:GetObject` |
| Uploads | Private | SSE-S3 encryption, versioning, CORS (PUT/GET from browser) |
| Deployments | Private | SSE-S3 encryption, versioning |

#### S3 VPC Gateway Endpoint

Before creating the buckets, we add a Gateway endpoint so S3 traffic from private subnets stays on the AWS backbone:

```hcl
data "aws_prefix_list" "s3" {
  name = "com.amazonaws.${var.region}.s3"
}

resource "aws_vpc_endpoint" "s3" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${var.region}.s3"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = [aws_route_table.private_az1.id, aws_route_table.private_az2.id]
  tags              = { Name = "s3-endpoint" }
}
```

Gateway endpoints are free — they work by adding a route to the route table. Any traffic destined for S3's IP ranges goes through the endpoint, bypassing the NAT Gateway and the public internet. The `data "aws_prefix_list"` query lets security group rules reference S3 by its managed prefix list later.

We also add the corresponding egress rule to the EC2 security group:

```hcl
resource "aws_security_group_rule" "ec2_egress_s3" {
  type              = "egress"
  security_group_id = aws_security_group.ec2_sg.id
  from_port         = 443
  to_port           = 443
  protocol          = "tcp"
  prefix_list_ids   = [data.aws_prefix_list.s3.id]
  description       = "HTTPS to S3 via VPC endpoint"
}
```

#### Frontend Bucket

Private bucket used for storing frontend build backups and artifacts (web hosting is handled by **AWS Amplify**):

```hcl
resource "aws_s3_bucket" "frontend" {
  bucket        = var.frontend_bucket_name
  force_destroy = true
  tags          = { Name = "frontend", Purpose = "frontend-artifacts" }
}

resource "aws_s3_bucket_public_access_block" "frontend" {
  bucket                  = aws_s3_bucket.frontend.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

Full `block_public_*` enabled — public access is blocked. Frontend static assets are served securely via **AWS Amplify Hosting**.


#### Uploads Bucket

Users upload files through the app. Private, encrypted, versioned, with CORS for browser uploads:

```hcl
resource "aws_s3_bucket" "uploads" {
  bucket        = var.uploads_bucket_name
  force_destroy = true
  tags          = { Name = "uploads", Purpose = "app-file-storage" }
}

resource "aws_s3_bucket_public_access_block" "uploads" {
  bucket                  = aws_s3_bucket.uploads.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_server_side_encryption_configuration" "uploads" {
  bucket = aws_s3_bucket.uploads.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_versioning" "uploads" {
  bucket = aws_s3_bucket.uploads.id
  versioning_configuration { status = "Enabled" }
}

resource "aws_s3_bucket_cors_configuration" "uploads" {
  bucket = aws_s3_bucket.uploads.id
  cors_rule {
    allowed_headers = ["*"]
    allowed_methods = ["PUT", "GET", "HEAD"]
    allowed_origins = ["*"]
    expose_headers  = ["ETag"]
    max_age_seconds = 3000
  }
}
```

All four `block_public_*` set to `true` — no accidental public exposure. CORS allows the browser to upload directly using pre-signed URLs from the backend.

#### Deployments Bucket

Stores backend deployment artifacts pulled by EC2 via SSM. Private, encrypted, versioned:

```hcl
resource "aws_s3_bucket" "deployments" {
  bucket        = var.deployments_bucket_name
  force_destroy = true
  tags          = { Name = "deployments", Purpose = "backend-code-artifacts" }
}

resource "aws_s3_bucket_public_access_block" "deployments" {
  bucket                  = aws_s3_bucket.deployments.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_server_side_encryption_configuration" "deployments" {
  bucket = aws_s3_bucket.deployments.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_versioning" "deployments" {
  bucket = aws_s3_bucket.deployments.id
  versioning_configuration { status = "Enabled" }
}
```