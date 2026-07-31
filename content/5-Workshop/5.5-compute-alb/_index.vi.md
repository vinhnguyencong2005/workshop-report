---
title : "Máy chủ & Cân bằng tải (Compute & Load Balancing)"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

Trong phần này, hệ thống triển khai hạ tầng xử lý tính toán và cân bằng tải nhằm đảm bảo ứng dụng Node.js backend hoạt động ổn định, tự động co giãn và an toàn. Các thành phần chính bao gồm:

| Thành phần | Chức năng |
|-----------|-------------|
| **Launch Template** | Bản thiết kế máy chủ EC2: AMI, instance type, script bootstrap user data, IAM profile, EBS volumes, IMDSv2 |
| **SSM VPC Endpoints** | Cho phép truy cập shell qua Session Manager mà không cần SSH hay bastion host |
| **Auto Scaling Group** | Duy trì 2–4 máy chủ trên hai AZs, tự động thay thế máy chủ lỗi, phân phối đều trên các private app subnets |
| **Application Load Balancer** | Điểm tiếp nhận lưu lượng HTTP :80, chuyển tiếp đến EC2 :3000, kiểm tra sức khỏe tại `/health` |
| **API Gateway HTTP API** | Proxy quản lý HTTPS tự động cho ALB, khắc phục triệt để lỗi Mixed Content trên trình duyệt |

#### Nội dung

1. [Launch Template](5.5.1-launch-template/) — Bản thiết kế EC2 với user data và IMDSv2
2. [SSM VPC Endpoints](5.5.2-ssm-endpoints/) — Truy cập shell an toàn không cần SSH
3. [Auto Scaling Group](5.5.3-asg/) — Tự động co giãn 2–4 máy chủ trên hai AZs
4. [Application Load Balancer](5.5.4-alb/) — Cân bằng tải với health check tự động
5. [API Gateway HTTP API](5.5.5-apigateway/) — Endpoint HTTPS cho ALB giải quyết lỗi Mixed Content
