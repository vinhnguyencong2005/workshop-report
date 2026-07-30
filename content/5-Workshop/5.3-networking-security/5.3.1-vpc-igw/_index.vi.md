---
title : "VPC & Internet Gateway"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.3.1. </b> "
---

Hạ tầng áp dụng một VPC duy nhất với dải mạng `10.0.0.0/16` — octet thứ ba được phân chia rõ ràng giữa ba tầng:

- `10.0.1–2` → Tầng Public (ALB, NAT Gateways)
- `10.0.10–11` → Tầng Private App (EC2)
- `10.0.20–21` → Tầng Private DB (RDS, Redis)

Khoảng trống (`10.0.3–9`, `10.0.12–19`) cho phép mở rộng các subnet trong tương lai mà không cần tái cấu trúc.

Cả hai cấu hình DNS đều được bật để EC2 có thể phân giải hostname nội bộ và các endpoint dịch vụ AWS.

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

Internet Gateway gắn trực tiếp vào VPC cung cấp đường truyền ra internet cho các public subnet. IGW không gán trực tiếp cho từng subnet — việc định tuyến được kiểm soát thông qua route table.

```hcl
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-igw"
  }
}
```
