---
title : "Tổng quan Workshop"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Giới thiệu

Trong workshop này, một ứng dụng **Hệ thống Quản lý Học tập (LMS) Full-stack** sẽ được triển khai trên AWS — hoàn toàn tự động bằng **Terraform**. Thay vì thao tác thủ công trên AWS Management Console, toàn bộ tài nguyên hạ tầng đều được định nghĩa dưới dạng mã nguồn (Infrastructure as Code): mạng, máy chủ tính toán, cơ sở dữ liệu, bộ nhớ đệm, lưu trữ, bảo mật và giám sát. Kết thúc workshop, lệnh `terraform destroy` sẽ dọn dẹp sạch sẽ toàn bộ tài nguyên.

Đây là hướng dẫn thực hành từng bước từ một tài khoản AWS trống đến một ứng dụng web quy mô doanh nghiệp, có khả năng mở rộng — không yêu cầu kinh nghiệm Terraform từ trước.

![Sơ đồ kiến trúc](/images/workshop/aws_architecture.png)

**Kiến trúc 3 tầng (3-tier architecture)** trải dài trên hai Availability Zones (AZs):

- **Tầng 1 (Public & Edge):** AWS Amplify Hosting phục vụ giao diện frontend React/Vite qua CDN toàn cầu. AWS API Gateway HTTP API đóng vai trò proxy HTTPS quản lý chuyển tiếp traffic tới Application Load Balancer. Regional AWS WAF v2 bảo vệ ALB trước các đợt tấn công web.
- **Tầng 2 (Private App):** Các máy chủ EC2 chạy Node.js với PM2 nằm trong Auto Scaling Group tại private subnets. Không sử dụng IP công khai. Lưu lượng truy cập ra internet đi qua NAT Gateways. Lưu lượng đầu vào **chỉ** chấp nhận từ ALB.
- **Tầng 3 (Private DB):** RDS MySQL (Multi-AZ), ElastiCache Redis và các bảng DynamoDB nằm trong các subnets cách ly hoàn toàn. Chỉ có tầng ứng dụng (App Tier) mới có thể kết nối tới các cơ sở dữ liệu này.

Các dịch vụ AWS được sử dụng trong kiến trúc:

| Nhóm dịch vụ | Dịch vụ AWS | Vai trò trong ứng dụng |
|----------|---------|--------------------------|
| **Frontend Hosting** | AWS Amplify Hosting | Host ứng dụng React/Vite SPA với CDN toàn cầu và quản lý SSL tự động |
| **API Gateway** | AWS API Gateway HTTP API | Endpoint HTTPS proxy quản lý chuyển tiếp lưu lượng tới ALB |
| **Edge Security** | Regional AWS WAF v2 | Web ACL bảo vệ ALB khỏi OWASP Top 10, bad inputs và giới hạn tần suất truy cập |
| **Compute** | EC2 Auto Scaling | Chạy ứng dụng backend Node.js; tự động thay thế máy chủ gặp sự cố |
| **Load Balancing** | Application Load Balancer | Phân phối lưu lượng tới các máy chủ EC2 trong private subnets; kiểm tra health check |
| **Cơ sở dữ liệu Quan hệ** | RDS MySQL (Multi-AZ) | Lưu trữ người dùng, khóa học, đăng ký học, điểm số |
| **Bộ nhớ đệm (Cache)** | ElastiCache Redis | Lưu trữ session, rate limiting, truy xuất nhanh |
| **Cơ sở dữ liệu NoSQL** | DynamoDB | Nội dung khóa học, bài đăng thảo luận, bài kiểm tra, lịch trình |
| **Lưu trữ Đối tượng** | Amazon S3 (3 buckets) | Lưu trữ riêng tư cho tệp tải lên, bản đóng gói ứng dụng, kết nối qua VPC Endpoints |
| **Quản lý & Giám sát** | CloudWatch + SSM | Cảnh báo CloudWatch (lỗi ALB, CPU, auto-scaling); SSM Session Manager truy cập máy chủ an toàn |

#### Tại sao chọn Terraform?

| Lý do | Ý nghĩa |
|--------|-----------------------|
| **Khai báo (Declarative)** | Định nghĩa *kết quả* mong muốn; Terraform tự tính toán *cách thức* khởi tạo |
| **Quy trình đơn giản** | `init → plan → apply → destroy` — bốn lệnh cho toàn bộ hạ tầng |
| **Theo dõi trạng thái** | Terraform theo dõi trạng thái thực tế và phát hiện sai lệch cấu hình (drift) |
| **Dọn dẹp triệt để** | `terraform destroy` xóa sạch mọi tài nguyên trong vài phút — không để lại tài nguyên rác |
| **Kỹ năng tái sử dụng** | Các mô hình học được (variables, outputs, modules) áp dụng cho mọi dự án cloud |

#### Kết quả đạt được

Mục tiêu hoàn thành sau workshop:

- Tạo IAM user và khởi tạo AWS access keys
- Cài đặt và cấu hình Terraform + AWS CLI
- Thiết kế VPC 3 tầng với các subnet Public, Private, và Database trên hai AZs
- Định tuyến lưu lượng với Internet Gateway, NAT Gateways và VPC Gateway Endpoints
- Cấu hình Security Groups tuân thủ nguyên tắc quyền tối thiểu (least privilege)
- Gán IAM roles cho máy chủ EC2 truy cập S3, DynamoDB, RDS và ElastiCache không cần hardcode credentials
- Triển khai Regional AWS WAF v2 Web ACL với các luật OWASP và rate limiting
- Cấu hình AWS Amplify Hosting cho ứng dụng React/Vite Single Page Application
- Triển khai AWS API Gateway HTTP API làm HTTPS proxy quản lý cho ALB
- Khởi tạo RDS MySQL Multi-AZ đảm bảo tính sẵn sàng cao
- Khởi tạo ElastiCache Redis hỗ trợ mã hóa dữ liệu (at-rest và in-transit)
- Thiết kế các bảng DynamoDB với Global Secondary Indexes cho truy vấn linh hoạt
- Tạo EC2 Auto Scaling Group kết hợp Launch Template và script khởi tạo `user_data`
- Định tuyến lưu lượng ứng dụng qua Application Load Balancer và cấu hình health checks
- Giám sát toàn bộ hệ thống bằng CloudWatch metric alarms và thông báo SNS
- Kết nối an toàn vào các máy chủ EC2 riêng tư qua SSM Session Manager
- Cấu hình quy trình tự động hóa CI/CD với GitHub Actions
- Dọn dẹp toàn bộ tài nguyên hạ tầng chỉ với một câu lệnh

#### Nguyên tắc Kiến trúc

{{% notice info %}}
Trong suốt workshop, kiến trúc tuân thủ các nguyên tắc cốt lõi:

- **Quyền tối thiểu (Least privilege):** Mỗi quy tắc Security Group và IAM policy chỉ cấp *đúng* quyền cần thiết.
- **Tính sẵn sàng cao (High availability):** Phân bổ trên hai Availability Zones cho ALB, EC2, RDS và Redis.
- **Không công khai IP cho máy chủ & CSDL:** EC2, RDS và Redis nằm hoàn toàn trong private subnets. Chỉ có ALB và Amplify tiếp xúc với internet.
- **Mã hóa toàn diện:** RDS, Redis, S3 và EBS volumes đều được mã hóa. Bắt buộc dùng IMDSv2 cho EC2.
- **Không sử dụng SSH:** SSM Session Manager cung cấp quyền truy cập shell an toàn mà không cần mở port 22.
{{% /notice %}}

#### Thời gian Ước tính

| Giai đoạn | Hoạt động | Thời gian |
|-------|----------|------|
| Thiết lập | Cài đặt Terraform, AWS CLI, Git; cấu hình IAM credentials; `terraform init` | 15 phút |
| Mạng & Bảo mật | VPC, 6 subnets trên 2 AZs, IGW, NAT GWs, route tables, VPC Gateway Endpoints, Security Groups, IAM Roles, Regional WAF v2 | 20 phút |
| Tầng Dữ liệu | S3 buckets, RDS MySQL Multi-AZ, ElastiCache Redis, DynamoDB tables, AWS Amplify Hosting | 20 phút |
| Máy chủ & ALB | Launch template, Auto Scaling Group, Application Load Balancer, API Gateway HTTP API | 20 phút |
| Giám sát | SNS topic, CloudWatch metric alarms (ALB 5xx, RDS CPU >80%, ASG below min) | 10 phút |
| Triển khai & Kiểm tra | `terraform apply`, kiểm tra kết nối qua SSM Session Manager, kiểm tra health check endpoints | 15 phút |
| CI/CD | Cấu hình GitHub Actions secrets, tự động deploy frontend lên Amplify và rolling refresh backend | 15 phút |
| Dọn dẹp | `terraform destroy`, xác nhận dọn dẹp sạch sẽ tài nguyên | 10 phút |
| **Tổng cộng** | | **~2 giờ** |

#### Nội dung Workshop

1. [Tổng quan Workshop](5.1-Workshop-overview/) ← Phần hiện tại
2. [Yêu cầu tiên quyết](5.2-Prerequiste/)
3. [Mạng & Bảo mật](5.3-networking-security/)
4. [Tầng Dữ liệu](5.4-data-layer/)
5. [Máy chủ & Cân bằng tải](5.5-compute-alb/)
6. [Giám sát & Cảnh báo](5.6-monitoring/)
7. [Triển khai & Kiểm tra](5.7-deploy-verify/)
8. [Quy trình CI/CD](5.8-cicd/)
9. [Dọn dẹp Tài nguyên](5.9-cleanup/)
