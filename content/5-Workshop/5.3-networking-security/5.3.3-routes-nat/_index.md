---
title : "Route Tables & NAT Gateways"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3.3. </b> "
---

Three route tables:

| Route Table | Attached To | `0.0.0.0/0` → |
|-------------|-------------|----------------|
| `public-rt` | `public_1`, `public_2` | IGW |
| `private-rt-az1` | `private_1`, `private_3` | NAT GW in AZ1 |
| `private-rt-az2` | `private_2`, `private_4` | NAT GW in AZ2 |

One public table shared by both public subnets. Two private tables — **one per AZ** — so each AZ has its own NAT Gateway. If AZ1 fails, AZ2 private instances still have internet access through their own NAT GW.

```hcl
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  tags = { Name = "public-rt" }
}

resource "aws_route_table_association" "public_1" {
  subnet_id      = aws_subnet.public_1.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public_2" {
  subnet_id      = aws_subnet.public_2.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table" "private_az1" {
  vpc_id = aws_vpc.main.id
  tags   = { Name = "private-rt-az1" }
}

resource "aws_route_table" "private_az2" {
  vpc_id = aws_vpc.main.id
  tags   = { Name = "private-rt-az2" }
}

resource "aws_route_table_association" "private_1" {
  subnet_id      = aws_subnet.private_1.id
  route_table_id = aws_route_table.private_az1.id
}

resource "aws_route_table_association" "private_2" {
  subnet_id      = aws_subnet.private_2.id
  route_table_id = aws_route_table.private_az2.id
}

resource "aws_route_table_association" "private_3" {
  subnet_id      = aws_subnet.private_3.id
  route_table_id = aws_route_table.private_az1.id
}

resource "aws_route_table_association" "private_4" {
  subnet_id      = aws_subnet.private_4.id
  route_table_id = aws_route_table.private_az2.id
}
```

Each NAT Gateway needs an Elastic IP, sits in a public subnet, and gets referenced by a route in the private route table:

```hcl
resource "aws_eip" "nat_az1" {
  domain = "vpc"
  tags   = { Name = "nat-eip-az1" }
}

resource "aws_eip" "nat_az2" {
  domain = "vpc"
  tags   = { Name = "nat-eip-az2" }
}

resource "aws_nat_gateway" "az1" {
  allocation_id = aws_eip.nat_az1.id
  subnet_id     = aws_subnet.public_1.id
  tags          = { Name = "nat-gw-az1" }
  depends_on    = [aws_internet_gateway.main]
}

resource "aws_nat_gateway" "az2" {
  allocation_id = aws_eip.nat_az2.id
  subnet_id     = aws_subnet.public_2.id
  tags          = { Name = "nat-gw-az2" }
  depends_on    = [aws_internet_gateway.main]
}

resource "aws_route" "private_az1_nat" {
  route_table_id         = aws_route_table.private_az1.id
  destination_cidr_block = "0.0.0.0/0"
  nat_gateway_id         = aws_nat_gateway.az1.id
}

resource "aws_route" "private_az2_nat" {
  route_table_id         = aws_route_table.private_az2.id
  destination_cidr_block = "0.0.0.0/0"
  nat_gateway_id         = aws_nat_gateway.az2.id
}
```

`depends_on` ensures the IGW exists before the NAT GW — the NAT GW needs internet access to function.

{{% notice warning %}}
Two NAT Gateways = ~$65/month. Worth it for production HA; a single NAT GW cuts cost in half if you can tolerate the AZ risk.
{{% /notice %}}
