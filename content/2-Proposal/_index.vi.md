---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
includeInReport: true
---

# Hạ tầng LMS 3 Tầng Doanh nghiệp trên AWS với Terraform
## Tự động hóa, Bảo mật và Mở rộng Hạ tầng Cloud cùng Quy trình CI/CD

### 1. Tóm tắt
Bản đề xuất này trình bày chi tiết việc thiết kế và triển khai tự động hạ tầng cloud 3 tầng đạt chuẩn doanh nghiệp trên Amazon Web Services (AWS) cho **Hệ thống Quản lý Học tập (LMS)**. Sử dụng **Terraform Infrastructure as Code (IaC)**, giải pháp tự động khởi tạo môi trường có tính sẵn sàng cao, bảo mật và khả năng mở rộng linh hoạt, phục vụ ứng dụng web tĩnh React/Vite (Single Page Application) cùng hệ thống API backend Node.js/Express cho sinh viên và giảng viên đại học.

Giải pháp tích hợp **AWS Amplify Hosting** để phân phối ứng dụng frontend toàn cầu, **AWS API Gateway HTTP API** làm proxy đảo HTTPS quản lý, **Regional AWS WAF v2** bảo mật ứng dụng web, **Application Load Balancer (ALB)** kết hợp **EC2 Auto Scaling Group (ASG)** trong các private subnet, **RDS MySQL (Multi-AZ)**, **ElastiCache Redis**, **DynamoDB**, và quy trình tự động hóa **GitHub Actions CI/CD pipelines**.

---

### 2. Chủ đề

#### Vấn đề hiện tại
1. **Khởi tạo Thủ công & Khó Nhân bản**: Việc khởi tạo máy chủ thủ công trên giao diện AWS Management Console dễ dẫn đến sai lệch cấu hình, lỗi con người và mất nhiều thời gian khi cần nhân bản môi trường.
2. **Rủi ro Bảo mật**: Đặt các máy chủ EC2 trực tiếp trong public subnet và mở port SSH 22 tiềm ẩn nguy cơ tấn công rất lớn. Ngoài ra, kết nối HTTP trực tiếp gây ra lỗi **Mixed Content** bị trình duyệt chặn khi frontend gọi từ HTTPS.
3. **Điểm Lỗi Đơn lẻ (Single Point of Failure)**: Mô hình máy chủ đơn lẻ dễ bị gián đoạn dịch vụ khi lưu lượng truy cập tăng đột biến hoặc máy chủ gặp sự cố phần cứng.
4. **Gián đoạn khi Cập nhật Code**: Cập nhật code thủ công yêu cầu dừng máy chủ, gây gián đoạn cho sinh viên khi đang làm bài thi hoặc xem tài liệu.

#### Giải pháp Đề xuất
Đề xuất giải pháp tự động hóa bằng **Infrastructure as Code (IaC)** xây dựng trên **Terraform**:
- **Không Lộ IP Công khai cho Máy chủ**: Các máy chủ EC2 nằm hoàn toàn trong private subnet và không có IP công khai. Quản trị máy chủ thông qua **AWS SSM Session Manager** (không cần quản lý khóa SSH hay mở port 22).
- **Bảo mật Biên & Gateway Quản lý**: **AWS WAF v2 (Regional)** bảo vệ ALB trước các tấn công OWASP Top 10, bad inputs và DDoS rate-limiting. **AWS API Gateway** cung cấp endpoint HTTPS tự động (`https://<api-id>.execute-api.us-east-1.amazonaws.com`) loại bỏ hoàn toàn lỗi Mixed Content.
- **Hosting Frontend Toàn cầu**: **AWS Amplify Hosting** phục vụ giao diện React/Vite qua mạng lưới CDN toàn cầu với quản lý chứng chỉ SSL tự động.
- **Tính Sẵn sàng Cao & Mở rộng Tự động**: RDS MySQL Multi-AZ, ElastiCache Redis cluster, và EC2 Auto Scaling Group tự động co giãn theo tải CPU.
- **Tự động hóa CI/CD**: GitHub Actions tự động deploy frontend lên Amplify và thực thi rolling instance refresh không gián đoạn dịch vụ cho backend.

---

#### Dịch vụ AWS & Modules Terraform Chính
- **Mạng & Bảo mật**: VPC, Public/Private/Database Subnets trên 2 AZs, Internet Gateway, NAT Gateways, Security Groups, IAM Roles (`vpc.tf`, `security_groups.tf`, `iam.tf`).
- **Web Application Firewall**: Regional AWS WAF v2 Web ACL với các luật OWASP Top 10, Bad Inputs, IP Reputation và Rate Limiting (`waf.tf`).
- **Hosting Frontend**: AWS Amplify Hosting cho React/Vite SPA với quy tắc điều hướng SPA (`amplify.tf`).
- **API Gateway**: HTTP API làm proxy HTTPS quản lý cho ALB (`apigateway.tf`).
- **Tính toán & Cân bằng tải**: Application Load Balancer, Launch Template, EC2 Auto Scaling Group với SSM managed instance profile (`alb.tf`, `ec2.tf`).
- **Dữ liệu & Lưu trữ**: RDS MySQL Multi-AZ (`rds.tf`), ElastiCache Redis (`redis.tf`), DynamoDB 5 bảng (`dynamodb.tf`), S3 Private Buckets (`s3.tf`) kết hợp S3 & DynamoDB VPC Gateway Endpoints.
- **Giám sát & Ghi log**: CloudWatch Log Groups và CloudWatch Alarm theo dõi chỉ số CPU (`cloudwatch.tf`).

---

### 4. Giai đoạn Triển khai Kỹ thuật

| Giai đoạn | Mốc công việc / Nhiệm vụ | Thời gian |
| :--- | :--- | :--- |
| **Giai đoạn 1: Kiến trúc & Thiết kế IaC** | Nghiên cứu kiến trúc 3 tầng AWS, thiết kế cấu trúc module HCL, định nghĩa biến (`variables.tf`) | Tuần 1 - 2 |
| **Giai đoạn 2: Cấu hình VPC & Bảo mật** | Viết code Terraform cho VPC, subnets đa AZ, NAT Gateways, IAM roles, security groups, Regional WAF v2 và API Gateway | Tuần 3 |
| **Giai đoạn 3: Khởi tạo Cơ sở dữ liệu & Máy chủ** | Provision RDS MySQL Multi-AZ, ElastiCache Redis, DynamoDB tables, S3 private buckets với VPC Endpoints, EC2 Launch Template và ASG | Tuần 4 |
| **Giai đoạn 4: Amplify Frontend & Bootstrap Máy chủ** | Cấu hình `amplify.tf`, viết script `user_data.tftpl` cho EC2 (Node.js 20, PM2, S3 artifact pull), kiểm tra kết nối HTTP/HTTPS | Tuần 5 |
| **Giai đoạn 5: CI/CD Pipeline & Kiểm thử** | Xây dựng GitHub Actions workflows cho Amplify frontend release và ASG rolling refresh; chạy smoke tests và viết tài liệu | Tuần 6 |

---

### 5. Ước tính Ngân sách

Ước tính chi phí hàng tháng theo bảng giá AWS (region us-east-1), dự tính cho **40,000 registered users** (~8,000 DAU, ~7.2 triệu API requests/tháng, ~2 TB frontend bandwidth/tháng):

#### Tier 1 — Compute & Load Balancing

| Dịch vụ AWS | Chi tiết Cấu hình | Chi phí/tháng (USD) |
| :--- | :--- | :--- |
| **EC2 Auto Scaling** | 2–4 × `t3.medium` instances (trung bình 2.5 instances) | ~$75.92 |
| **EBS Storage (gp3)** | 30 GB/instance × 2.5 instances | ~$6.00 |
| **Application Load Balancer** | 1 ALB (fixed + ~2.5 LCU data processing) | ~$31.03 |
| **NAT Gateways** | 2 NAT Gateways (1/AZ, always-on) + ~50 GB processed | ~$67.95 |
| **Public IPv4 Addresses** | 4 IPs (2 NAT EIPs + 2 ALB IPs) | ~$14.60 |
| **VPC Gateway Endpoints** | S3 & DynamoDB (free of charge) | $0.00 |
| | **Tier 1 Subtotal** | **~$195.50** |

#### Tier 2 — Database & Storage

| Dịch vụ AWS | Chi tiết Cấu hình | Chi phí/tháng (USD) |
| :--- | :--- | :--- |
| **RDS MySQL Multi-AZ** | `db.t4g.micro`, 20 GB gp3 storage, 7-day backup | ~$25.66 |
| **ElastiCache Redis** | `cache.t4g.micro`, 2-node (primary + replica) | ~$23.36 |
| **DynamoDB (5 tables)** | On-Demand, PITR enabled (~12M writes + 48M reads/tháng) | ~$15.75 |
| **Amazon S3** | 3 private buckets, ~20 GB storage | ~$1.50 |
| | **Tier 2 Subtotal** | **~$66.27** |

#### Tier 3 — Security, Frontend Hosting & Monitoring

| Dịch vụ AWS | Chi tiết Cấu hình | Chi phí/tháng (USD) |
| :--- | :--- | :--- |
| **AWS WAF v2 (Regional)** | 1 Web ACL + 3 Managed Rule Groups + 1 Rate Limit rule + 7.2M requests | ~$13.32 |
| **AWS Amplify Hosting** | React/Vite SPA, ~2,000 GB bandwidth @ $0.15/GB | ~$300.00 |
| **API Gateway HTTP API** | 7.2M requests @ $1.00/1M | ~$7.20 |
| **VPC Interface Endpoints (SSM)** | 3 endpoints × 2 AZs = 6 ENIs @ $0.01/hr | ~$43.80 |
| **CloudWatch** | 3 alarms + ~10 GB log ingestion | ~$5.30 |
| **IAM** | Roles & Policies | $0.00 |
| | **Tier 3 Subtotal** | **~$369.62** |

#### Tổng Chi phí Ước tính Hàng tháng

| | Chi phí/tháng (USD) |
| :--- | :--- |
| Tier 1 — Compute & Load Balancing | $195.50 |
| Tier 2 — Database & Storage | $66.27 |
| Tier 3 — Security, Frontend & Monitoring | $369.62 |
| **Tổng Chi phí Ước tính** | **~$631.39 / tháng** |

*Lưu ý: Chi phí lớn nhất là Amplify Hosting bandwidth (~$300, chiếm 47.5% tổng chi phí). Chuyển sang CloudFront + S3 static hosting có thể giảm bandwidth cost xuống ~$170/tháng. Compute Savings Plans và Reserved Instances cho EC2/RDS/Redis có thể tiết kiệm thêm 30–50%. Khi thực hành workshop, chi phí có thể giảm về gần $0 bằng cách tear down tài nguyên với `terraform destroy`.*

---

### 6. Đánh giá Rủi ro & Chiến lược Giảm thiểu

| Rủi ro Ghi nhận | Mức độ Ảnh hưởng | Xác suất | Chiến lược Giảm thiểu |
| :--- | :--- | :--- | :--- |
| **Lỗi Bảo mật Mixed Content trên Trình duyệt** | Cao | Cao | Triển khai **AWS API Gateway HTTP API** cung cấp endpoint HTTPS, chuyển tiếp an toàn tới ALB. |
| **Tấn công Tấn công DDoS hoặc Bot Độc hại** | Cao | Trung bình | Khởi tạo **Regional AWS WAF v2** với các luật giới hạn tần suất (2,000 req/5phút) và OWASP Top 10. |
| **Máy chủ EC2 Cập nhật Lỗi hoặc Quá tải** | Cao | Trung bình | Cấu hình **EC2 Auto Scaling Group** kết hợp CloudWatch alarm tự động co giãn và kiểm tra health check. |
| **Chi phí Phát sinh Ngoài ý muốn** | Trung bình | Thấp | Cấu hình CloudWatch billing alert và script tự động xóa hạ tầng bằng `terraform destroy`. |

---

### 7. Kết quả Kỳ vọng

1. **100% Infrastructure as Code**: Khởi tạo toàn bộ hạ tầng cloud doanh nghiệp trong khoảng ~15 phút bằng `terraform apply`.
2. **Quy trình CI/CD Không Gián đoạn**: Tự động hóa hoàn toàn quy trình deploy cho Amplify frontend và ASG rolling instance refresh backend.
3. **Tiêu chuẩn Bảo mật Cao**: EC2 nằm hoàn toàn trong private subnet, truy cập máy chủ qua SSM Session Manager, bảo vệ bởi Regional WAF và HTTPS toàn trình.
4. **Bộ Tài liệu Workshop Hoàn chỉnh**: Tài liệu workshop Hugo và nhật ký công việc (worklog) chi tiết, sẵn sàng cho việc mở rộng nhóm và làm tài liệu tham khảo học thuật.