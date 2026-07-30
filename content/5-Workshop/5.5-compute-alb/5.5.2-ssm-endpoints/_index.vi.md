---
title : "SSM VPC Endpoints"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.5.2. </b> "
---

Ba Interface VPC Endpoints giúp kích hoạt AWS Systems Manager Session Manager — cho phép truy cập shell trên các máy chủ EC2 private mà không cần SSH, bastion host hay mở các cổng kết nối đầu vào.

```hcl
resource "aws_vpc_endpoint" "ssm" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.ssm"
  vpc_endpoint_type   = "Interface"
  private_dns_enabled = true
  subnet_ids          = [aws_subnet.private_1.id, aws_subnet.private_2.id]
  security_group_ids  = [aws_security_group.vpce_sg.id]
  tags                = { Name = "ssm-endpoint" }
}

resource "aws_vpc_endpoint" "ssmmessages" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.ssmmessages"
  vpc_endpoint_type   = "Interface"
  private_dns_enabled = true
  subnet_ids          = [aws_subnet.private_1.id, aws_subnet.private_2.id]
  security_group_ids  = [aws_security_group.vpce_sg.id]
  tags                = { Name = "ssmmessages-endpoint" }
}

resource "aws_vpc_endpoint" "ec2messages" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.ec2messages"
  vpc_endpoint_type   = "Interface"
  private_dns_enabled = true
  subnet_ids          = [aws_subnet.private_1.id, aws_subnet.private_2.id]
  security_group_ids  = [aws_security_group.vpce_sg.id]
  tags                = { Name = "ec2messages-endpoint" }
}
```

| Endpoint | Chức năng |
|----------|---------|
| `ssm` | API SSM cốt lõi — đăng ký máy chủ, tiếp nhận yêu cầu phiên làm việc |
| `ssmmessages` | Kênh truyền dữ liệu phiên — truyền tải dữ liệu terminal input/output |
| `ec2messages` | Giao tiếp với SSM Agent — gửi lệnh và cập nhật trạng thái |

Cả ba đều là **Interface** endpoints — tự động tạo các ENI với IP private trong app subnets. Cấu hình `private_dns_enabled = true` giúp SSM Agent tự động phân giải DNS của dịch vụ SSM về IP private endpoint.

Tất cả endpoints sử dụng security group `vpce-sg` (cho phép HTTPS inbound từ dải VPC CIDR). IAM Role của EC2 cần đính kèm policy `AmazonSSMManagedInstanceCore`.

Sau khi triển khai, việc kết nối vào máy chủ được thực hiện đơn giản qua:

```bash
aws ssm start-session --target i-0123456789abcdef
```

Không cần key pairs. Không cần cổng 22. Không cần máy chủ bastion.
