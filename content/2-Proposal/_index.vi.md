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

### 5. Ước tính Ngân sách & Khung Chi phí

Trong giai đoạn đề xuất ý tưởng ban đầu, chi phí cloud thực tế phụ thuộc vào lượng người dùng sử dụng, lưu lượng truy cập đỉnh điểm và cấu hình tài nguyên được lựa chọn. Dưới đây là khung ước tính chi phí linh hoạt được phân loại theo các tầng kiến trúc và quy mô vận hành.

#### Các Yếu tố Chi phí Chính

1. **Tính toán & Cân bằng tải (Tier 1)**: Máy chủ EC2 Auto Scaling (`t3.medium`/`t4g.small`), Application Load Balancer (ALB), và NAT Gateways.
2. **Cơ sở dữ liệu & Lưu trữ (Tier 2)**: RDS MySQL (Multi-AZ), ElastiCache (Redis), DynamoDB (On-Demand), và Amazon S3 storage.
3. **Bảo mật, Phân phối & Kết nối (Tier 3)**: Băng thông CDN (AWS Amplify / CloudFront), Regional AWS WAF v2, API Gateway HTTP API, và VPC Endpoints (SSM).

#### Khoảng Ngân sách Ước tính theo Quy mô Vận hành

| Môi trường / Quy mô | Khoảng Chi phí/Tháng | Cấu hình & Giả định Chính |
| :--- | :--- | :--- |
| **Thử nghiệm / Lab** | **~$0 – $30 / tháng** | Tài nguyên tạm thời được khởi tạo khi cần qua `terraform apply` và xóa bỏ bằng `terraform destroy` khi hoàn thành. Nằm phần lớn trong AWS Free Tier. |
| **Môi trường Staging / Quy mô Nhỏ** | **~$150 – $350 / tháng** | Cấu hình tối thiểu (`t4g.micro`), tối ưu số lượng NAT Gateway, lưu lượng CDN vừa phải (<500 GB/tháng). |
| **Quy mô Sản xuất (Production)** | **~$500 – $800 / tháng** | Đáp ứng tính sẵn sàng cao đa AZ (~8,000 daily active user, 40,000 sinh viên), tự động co giãn (2–4 instances), bảo mật WAF toàn diện và ~2 TB băng thông CDN. |

#### Chiến lược Quản lý & Tối ưu Chi phí

- **Tối ưu Băng thông**: Sử dụng AWS CloudFront + S3 static hosting làm phương án thay thế cho Amplify Hosting có thể giúp giảm chi phí truyền dữ liệu CDN tới 45%.
- **Cam kết Sử dụng (Savings Plans)**: Áp dụng Compute Savings Plans hoặc Reserved Instances (1–3 năm) cho EC2, RDS và ElastiCache giúp tiết kiệm từ 30%–50% chi phí cho môi trường chạy ổn định.
- **Tự động Xóa tài nguyên (Teardown)**: Với môi trường thử nghiệm/phát triển, việc sử dụng `terraform destroy` sau mỗi phiên làm việc giúp giảm chi phí về mốc tối thiểu.

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