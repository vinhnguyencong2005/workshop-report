---
title : "VPC & Internet Gateway"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.3.1. </b> "
---

We use a single VPC with `10.0.0.0/16` — chosen so the third octet cleanly separates our three tiers:

- `10.0.1–2` → Public (ALB, NAT Gateways)
- `10.0.10–11` → Private App (EC2)
- `10.0.20–21` → Private DB (RDS, Redis)

The gaps (`10.0.3–9`, `10.0.12–19`) leave room for future subnets without re-architecting.

Both DNS settings are enabled so EC2 can resolve internal hostnames and AWS service endpoints.

```hcl
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "main"
  }
}
```

A single Internet Gateway attached to the VPC gives public subnets a path to the internet. The IGW itself doesn't get assigned to specific subnets — we control that through route tables in the next sections.

```hcl
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-igw"
  }
}
```