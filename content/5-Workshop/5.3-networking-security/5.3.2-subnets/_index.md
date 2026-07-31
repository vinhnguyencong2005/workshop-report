---
title : "Subnets"
date: 2026-07-30
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

#### Six Subnets, Three Tiers

Six subnets across two AZs, organized into three tiers. Each tier has one subnet per AZ so the application survives a single-AZ failure:

If `us-east-1a` fails, everything in `us-east-1b` keeps running.

#### Public — for the ALB and NAT Gateways

The only distinction from private subnets: `map_public_ip_on_launch = true`.

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

#### Private App — for EC2 instances

No `map_public_ip_on_launch` — instances get no public IP. Outbound internet goes through NAT Gateways; the internet cannot initiate connections inbound.

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

#### Private DB — for RDS and Redis

Deepest isolation. No internet in either direction. The `Tier = "db"` tag is used later by RDS and ElastiCache subnet groups to select these subnets.

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
