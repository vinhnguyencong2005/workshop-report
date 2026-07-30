---
title : "Route Tables & NAT Gateways"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3.3. </b> "
---

Ba bảng định tuyến (Route Tables):

| Route Table | Gắn với Subnets | `0.0.0.0/0` → |
|-------------|-------------|----------------|
| `public-rt` | `public_1`, `public_2` | IGW |
| `private-rt-az1` | `private_1`, `private_3` | NAT GW thuộc AZ1 |
| `private-rt-az2` | `private_2`, `private_4` | NAT GW thuộc AZ2 |

Một route table public chung cho cả hai public subnets. Hai route table private — **mỗi AZ một bảng** — đảm bảo mỗi AZ sử dụng NAT Gateway riêng. Nếu AZ1 gặp sự cố, các private instances tại AZ2 vẫn duy trì kết nối internet thông qua NAT Gateway riêng.

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

Mỗi NAT Gateway sử dụng một IP tĩnh (Elastic IP), đặt trong public subnet và được tham chiếu bởi tuyến đường trong route table private:

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

Tham số `depends_on` đảm bảo IGW đã sẵn sàng trước khi tạo NAT GW.

{{% notice warning %}}
Hai NAT Gateways tiêu tốn khoảng ~$65/tháng. Đáng giá cho môi trường production HA; có thể giảm xuống 1 NAT Gateway để tiết kiệm 50% chi phí nếu chấp nhận rủi ro khi một AZ gặp sự cố.
{{% /notice %}}
