---
title : "S3 Buckets & VPC Endpoint"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

Ba buckets, mỗi bucket có hình thức truy cập khác nhau:

| Bucket | Bảo mật | Cấu hình chính |
|--------|---------|-------------|
| Frontend | Riêng tư (Private) | Lưu trữ bản sao lưu và artifact (Hosting do Amplify quản lý) |
| Uploads | Riêng tư (Private) | Mã hóa SSE-S3, versioning, CORS (cho phép PUT/GET từ trình duyệt) |
| Deployments | Riêng tư (Private) | Mã hóa SSE-S3, versioning |

#### S3 VPC Gateway Endpoint

Trước khi tạo các buckets, chúng ta thêm một Gateway endpoint để lưu lượng S3 từ các private subnets đi trên hạ tầng AWS backbone:

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

Gateway endpoints hoàn toàn miễn phí — chúng hoạt động bằng cách thêm tuyến đường vào route table. Bất kỳ lưu lượng nào đến dải IP của S3 đều đi qua endpoint mà không qua NAT Gateway hay Internet công cộng.

Chúng ta cũng thêm luật egress tương ứng cho security group của EC2:

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

Bucket riêng tư được dùng để lưu trữ các bản sao lưu build và artifact của ứng dụng frontend (việc hosting trang web được đảm nhiệm bởi **AWS Amplify**):

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

Toàn bộ tùy chọn `block_public_*` được bật `true` — chặn hoàn toàn truy cập công khai. Các tệp tĩnh tĩnh frontend được phục vụ an toàn qua **AWS Amplify Hosting**.

#### Uploads Bucket

Người dùng tải tệp lên thông qua ứng dụng. Bucket riêng tư, được mã hóa, bật versioning, và cấu hình CORS cho việc upload từ trình duyệt:

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

#### Deployments Bucket

Lưu trữ gói deployment backend được EC2 pull về thông qua SSM. Riêng tư, mã hóa, bật versioning:

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
