---
title : "Subnets"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

#### Sáu Subnets, Ba Tầng

Sáu subnets trải dài trên hai AZs, được tổ chức thành ba tầng. Mỗi tầng có một subnet trên mỗi AZ giúp hệ thống duy trì hoạt động khi một AZ gặp sự cố:

Nếu `us-east-1a` bị gián đoạn, toàn bộ dịch vụ trên `us-east-1b` vẫn tiếp tục hoạt động.

#### Public Subnets — Dành cho ALB và NAT Gateways

Điểm khác biệt duy nhất so với private subnets: `map_public_ip_on_launch = true`.

```hcl
resource "aws_subnet" "public_1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "public_1"
    Type = "public"
    AZ   = "us-east-1a"
  }
}

resource "aws_subnet" "public_2" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true

  tags = {
    Name = "public_2"
    Type = "public"
    AZ   = "us-east-1b"
  }
}
```

#### Private App Subnets — Dành cho các máy chủ EC2

Không có `map_public_ip_on_launch` — máy chủ không được cấp IP công khai. Lưu lượng outbound ra internet đi qua NAT Gateways; internet không thể chủ động kết nối vào các máy chủ này.

```hcl
resource "aws_subnet" "private_1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.10.0/24"
  availability_zone = "us-east-1a"

  tags = {
    Name = "private_1"
    Type = "private"
    AZ   = "us-east-1a"
    Tier = "app"
  }
}

resource "aws_subnet" "private_2" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.11.0/24"
  availability_zone = "us-east-1b"

  tags = {
    Name = "private_2"
    Type = "private"
    AZ   = "us-east-1b"
    Tier = "app"
  }
}
```

#### Private DB Subnets — Dành cho RDS và Redis

Mức độ cô lập cao nhất. Không kết nối internet hai chiều. Tag `Tier = "db"` được dùng sau đó cho RDS và ElastiCache subnet groups để chọn đúng các subnets này.

```hcl
resource "aws_subnet" "private_3" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.20.0/24"
  availability_zone = "us-east-1a"

  tags = {
    Name = "private_3"
    Type = "private"
    AZ   = "us-east-1a"
    Tier = "db"
  }
}

resource "aws_subnet" "private_4" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.21.0/24"
  availability_zone = "us-east-1b"

  tags = {
    Name = "private_4"
    Type = "private"
    AZ   = "us-east-1b"
    Tier = "db"
  }
}
```
