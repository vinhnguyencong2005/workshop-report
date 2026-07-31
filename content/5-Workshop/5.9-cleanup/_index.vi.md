---
title : "Dọn dẹp tài nguyên"
date: 2026-07-30
weight : 9
chapter : false
pre : " <b> 5.9. </b> "
---

Để tránh phát sinh chi phí ngoài ý muốn trên tài khoản AWS sau khi hoàn tất buổi workshop, việc dọn dẹp toàn bộ hạ tầng cloud đã tạo là vô cùng quan trọng.

Do mọi tài nguyên đã được khai báo tập trung bằng **Terraform Infrastructure as Code**, bạn có thể xóa toàn bộ hơn 50+ tài nguyên AWS một cách sạch sẽ chỉ bằng một lệnh duy nhất.

---

#### Bước 1: Xóa Hạ tầng (`terraform destroy`)

1. Mở terminal và truy cập vào thư mục Terraform:
   ```bash
   cd Terraform_
   ```

2. Chạy lệnh xóa tài nguyên:
   ```bash
   terraform destroy
   ```

3. Terraform sẽ liệt kê toàn bộ các tài nguyên sắp bị xóa. Gõ **`yes`** khi được hỏi để xác nhận:

![terraform destroy](/images/workshop/5.9/1.png)

##### Thời gian dọn dẹp dự kiến:
- **0–2 phút**: WAF Web ACL, API Gateway HTTP API, ALB listeners, Security Groups, IAM instance profiles
- **5-10 phút**: EC2 Auto Scaling Group, Launch Template, S3 buckets, DynamoDB tables, AWS Amplify App
- **5–12 phút**: RDS MySQL Multi-AZ DB Instance và ElastiCache Redis Replication Group
- **12–15 phút**: NAT Gateways, Elastic IPs, VPC subnets, Internet Gateway, VPC

Tổng thời gian dọn dẹp kéo dài khoảng **20–25 phút**.
