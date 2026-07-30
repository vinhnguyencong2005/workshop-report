---
title : "SSM VPC Endpoints"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.5.2. </b> "
---

Three Interface VPC Endpoints enable AWS Systems Manager Session Manager — our way to get a shell on private EC2 instances without SSH, bastion hosts, or open inbound ports.

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

| Endpoint | Purpose |
|----------|---------|
| `ssm` | Core SSM API — registers instances, accepts session requests |
| `ssmmessages` | Session data channel — carries terminal input/output |
| `ec2messages` | SSM Agent communication — commands, status updates |

All three are **Interface** endpoints — they create ENIs with private IPs in the app subnets. `private_dns_enabled = true` means the SSM Agent resolves the public SSM service DNS to the private endpoint IP automatically.

These endpoints use the `vpce-sg` security group created earlier (allows HTTPS inbound from the VPC CIDR). The EC2 IAM role also needs `AmazonSSMManagedInstanceCore` attached — already done in the IAM section.

Once deployed, connecting to an instance is:

```bash
aws ssm start-session --target i-0123456789abcdef
```

No key pairs. No port 22. No bastion.