---
title: "Worklog Tuần 6"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6

- Triển khai toàn bộ ứng dụng lên hạ tầng AWS với Terraform
- Thiết lập quy trình CI/CD tự động hóa deploy backend và frontend

### Các công việc cần triển khai trong tuần này

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | -------------- |
| 2   | - Chạy `terraform apply` khởi tạo hạ tầng ban đầu: VPC, public subnet, EC2 instances <br> - SSH vào instance, cài Node.js 20 qua nvm, clone source code backend thủ công <br> - Cấu hình `.env` với database credentials, JWT secret, Redis và AWS region <br> - Cài PM2 process manager, khởi động backend trên port 3000, kiểm tra qua public IP <br> - Ghi nhận các vấn đề ban đầu: hardcoded config, chưa có auto-restart, EC2 lộ trực tiếp ra internet | 20/07/2026 | 20/07/2026 | |
| 3   | - Chuyển đổi lưu trữ frontend sang **AWS Amplify Hosting** (`amplify.tf`) để có CDN toàn cầu và HTTPS SSL tự động <br> - Cấu hình SPA client-side rewrite rules trong `amplify.tf` cho điều hướng index.html <br> - Kiểm tra kết nối full-stack: frontend tải từ Amplify qua HTTPS <br> - Đánh giá bảo mật: EC2 trực tiếp trên internet là rủi ro lớn — bắt đầu nghiên cứu private subnet <br> - Thêm ALB, private subnet, và NAT Gateway vào cấu hình Terraform | 21/07/2026 | 21/07/2026 | |
| 4   | - Refactor Terraform: chuyển EC2 vào private subnet, định tuyến outbound qua NAT Gateway <br> - Tạo Application Load Balancer trong public subnet, forward đến EC2 target group port 3000 <br> - Khởi tạo **Regional AWS WAF v2** (`waf.tf`) gắn trực tiếp vào ALB với các AWS Managed Rules (OWASP Top 10, Bad Inputs, IP Reputation) và IP Rate Limiting <br> - Giải quyết bài toán deploy: private instance không có public IP — thiết kế chiến lược S3 + SSM <br> - Khởi tạo **API Gateway HTTP API** (`apigateway.tf`) cung cấp endpoint HTTPS quản lý cho ALB, khắc phục triệt để lỗi Mixed Content trên trình duyệt | 22/07/2026 | 22/07/2026 | |
| 5   | - Cải tiến kiến trúc deploy: thay thế SSM push bằng user_data pull-on-boot để deploy bất biến (immutable) <br> - Viết script `user_data.tftpl`: cài AWS CLI v2, nvm + Node.js 20, PM2, pull backend.zip từ S3, giải nén, `npm ci --production`, chạy app qua PM2 <br> - Provision RDS MySQL (Multi-AZ) và ElastiCache Redis (2-node cluster) qua Terraform <br> - Cấu hình các bảng DynamoDB (ClassContent, ForumData, QuizContent, StudentSchedule, CourseAssign) trên AWS <br> - Bảo mật S3 frontend bucket bằng cách tắt public website access và bật full public access block | 23/07/2026 | 23/07/2026 | |
| 6   | - Thiết lập GitHub Actions CI/CD workflow: frontend qua AWS Amplify CLI API (`create-deployment` + upload + `start-deployment`) và backend qua S3 + ASG Rolling Refresh <br> - Viết IAM policy cho EC2 instance role: quyền S3 read/write, DynamoDB CRUD, Secrets Manager, SSM managed instance <br> - Cấu hình security groups theo nguyên tắc least-privilege: EC2 chỉ nhận ingress từ ALB, RDS chỉ từ EC2, Redis chỉ từ EC2 <br> - Cấu hình luật WAF v2 rate limiting (2,000 req/5phút) và thiết lập CORS trên API Gateway | 24/07/2026 | 24/07/2026 | |
| 7   | - Chạy kiểm thử deploy tự động toàn trình: push code → GitHub Actions chạy → frontend deploy lên Amplify → ASG instance refresh rollout backend → health check pass <br> - Gỡ lỗi và sửa lỗi bảo mật Mixed Content: định tuyến API calls từ frontend qua endpoint API Gateway HTTPS (`https://<api-id>.execute-api.us-east-1.amazonaws.com`) <br> - Tinh chỉnh ASG rolling refresh: đặt `InstanceWarmup: 0` và interval poll 10s để CI/CD chạy cực nhanh <br> - Kiểm tra ASG auto-scaling: trigger scale-out thủ công, xác nhận instance mới pull code mới nhất và đăng ký với ALB <br> - Ghi chép tất cả lỗi gặp phải và cách khắc phục | 25/07/2026 | 25/07/2026 | |
| CN  | - Tổ chức lại cấu trúc thư mục Terraform: tách file cấu hình lớn thành các file riêng (`waf.tf`, `amplify.tf`, `apigateway.tf`, v.v.) <br> - Trích xuất các giá trị hardcoded thành biến (region, instance types, CIDR blocks) với `terraform.tfvars` <br> - Thay thế tên domain cá nhân mặc định bằng domain dùng chung `lms.uni` (`app.lms.uni`) <br> - Viết README hạ tầng và tài liệu workshop Hugo với sơ đồ kiến trúc và hướng dẫn triển khai <br> - Chạy smoke test production lần cuối: đăng ký → đăng nhập → xem khóa học → làm quiz → nộp bài → xem điểm <br> - Push toàn bộ code Terraform và ứng dụng lên GitHub, xác nhận CI/CD chạy khi push | 26/07/2026 | 26/07/2026 | |

### Kết quả đạt được sau tuần 6

- Hạ tầng AWS được tự động hóa hoàn toàn qua Terraform: VPC, ALB, Regional AWS WAF v2, AWS Amplify Hosting, API Gateway HTTP API, EC2 ASG trong private subnet, RDS MySQL Multi-AZ, ElastiCache Redis, DynamoDB (5 bảng), S3 (private buckets)
- Chiến lược deploy bất biến (immutable) đã thiết lập: backend qua S3 artifact + ASG rolling instance refresh, frontend qua AWS Amplify 3-step deployment API
- CI/CD pipeline hoạt động trên GitHub Actions: `git push main` tự động deploy frontend lên Amplify và backend không gián đoạn (zero-downtime)
- Baseline bảo mật chuyên sâu đã triển khai: Regional WAF v2 (OWASP Top 10, IP reputation, rate limiting), API Gateway HTTPS, private subnet, security groups và IAM roles least-privilege
- Lỗi bảo mật Mixed Content trên trình duyệt đã được khắc phục hoàn toàn bằng API Gateway HTTPS proxying đến ALB
- Tài liệu hạ tầng, hướng dẫn workshop Hugo và các mẫu biến đã sẵn sàng cho nhóm cộng tác

