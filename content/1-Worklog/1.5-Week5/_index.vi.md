---
title: "Worklog Tuần 5"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5

- Hoàn thiện thiết kế kiến trúc hệ thống cloud 3 tầng (3-Tier Architecture) cho toàn bộ dự án LMS.
- Thống nhất API Contract, chuẩn hóa mô hình dữ liệu (Database Schema) và cơ chế bảo mật/phân quyền để hỗ trợ Frontend và Backend devs.
- Triển khai Hạ tầng làm Mã (IaC với Terraform), API Gateway, WAF, S3 Presigned URL và thiết lập pipeline CI/CD.
- Theo dõi, đánh giá tiến độ và hỗ trợ giải quyết vướng mắc tích hợp (CORS, Environment variables, CloudWatch logs) cho lập trình viên Frontend và Backend.

### Các công việc triển khai trong tuần 5

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | -------------- |
| 2   | **Thiết kế Kiến trúc Hệ thống & Quy chuẩn API Contract**: <br> - Thiết kế kiến trúc tổng thể 3 tầng trên AWS (Amplify Frontend + ALB/ASG Backend + RDS/Redis/DynamoDB) <br> - Xây dựng tài liệu OpenAPI/Swagger định nghĩa chuẩn dữ liệu request/response giữa Frontend & Backend <br> - *Tiến độ nhóm*: Backend triển khai cơ bản Auth API; Frontend xây dựng UI layout khung ứng dụng | 13/07/2026 | 13/07/2026 | AWS Architecture Framework |
| 3   | **Thiết kế Cơ sở Dữ liệu & Giải pháp Lưu trữ**: <br> - Thiết kế mô hình cơ sở dữ liệu quan hệ RDS MySQL (Multi-AZ) và DynamoDB cho audit logs <br> - Thiết kế kiến trúc bộ nhớ đệm ElastiCache Redis cho session và rate-limiting <br> - Thiết kế giải pháp upload file tài liệu trực tiếp lên S3 sử dụng S3 Presigned URL (tránh quá tải EC2) <br> - *Tiến độ nhóm*: Backend hoàn thiện các ORM model; Frontend dựng trang quản lý khóa học và bài giảng | 14/07/2026 | 14/07/2026 | AWS RDS & S3 Best Practices |
| 4   | **Cấu hình Mạng, Gateway & Bảo mật Ứng dụng**: <br> - Thiết kế cấu trúc VPC (Public/Private subnets), Security Groups và IAM Roles chuẩn least-privilege <br> - Cấu hình API Gateway HTTP API làm reverse proxy HTTPS và Regional WAF v2 chống tấn công OWASP Top 10 <br> - Cấu hình chuẩn hóa CORS headers và SSL/TLS để loại bỏ hoàn toàn lỗi Mixed Content cho Frontend <br> - *Tiến độ nhóm*: Backend tích hợp API Quiz & Judge0 chấm code; Frontend ghép nối API đăng nhập và danh sách lớp học | 15/07/2026 | 15/07/2026 | AWS WAF & API Gateway Docs |
| 5   | **Viết Mã Hạ tầng (IaC Terraform) & Tự động hóa CI/CD**: <br> - Đóng gói toàn bộ tài nguyên cloud thành các Terraform module (`vpc.tf`, `alb.tf`, `ec2.tf`, `amplify.tf`) <br> - Xây dựng script bootstrap EC2 (`user_data.tftpl`) cài đặt Node.js 20, PM2 và S3 artifact sync <br> - Xây dựng quy trình CI/CD GitHub Actions tự động build/deploy Frontend lên Amplify và Backend lên ASG <br> - *Tiến độ nhóm*: Backend hoàn thiện các API còn lại; Frontend hoàn thành UI làm bài Quiz và Submit Code | 16/07/2026 | 16/07/2026 | Terraform AWS Provider |
| 6   | **Kiểm thử Tích hợp, Giám sát hệ thống & Họp Đánh giá Tiến độ**: <br> - Thiết lập CloudWatch Log Groups và Alarms theo dõi chỉ số CPU/bộ nhớ, giúp devs debug lỗi runtime <br> - Tổ chức phiên kiểm thử tích hợp (End-to-End integration test) giữa Frontend và Backend <br> - Review PR code, tối ưu hóa truy vấn DB và xử lý các lỗi phát sinh trong môi trường dev/staging <br> - Họp tổng kết tiến độ tuần 5 và lên kế hoạch kiểm thử tải (Load Testing) cho tuần 6 | 17/07/2026 | 17/07/2026 | Internal Dev Guidelines |

### Kết quả đạt được sau tuần 5

- **Kiến trúc & Hạ tầng**: Hoàn thiện 100% sơ đồ và mã Terraform cho hạ tầng 3 tầng trên AWS (VPC, WAF, API Gateway, ALB, ASG, RDS, Redis, DynamoDB, S3).
- **Hỗ trợ Đội ngũ Phát triển**:
  - Chuẩn hóa tài liệu API Contract giúp Frontend và Backend dev làm việc song song không bị nghẽn (unblock).
  - Khắc phục triệt để lỗi CORS và Mixed Content khi Frontend gọi API HTTPS.
  - Thiết lập kênh theo dõi log tập trung qua CloudWatch giúp Backend devs debug sự cố nhanh chóng.
- **Tiến độ Dự án**:
  - **Backend**: Hoàn thành 100% các nhóm API (Auth, Courses, Forum, Materials, Quizzes, Judge0 Code Execution).
  - **Frontend**: Ghép nối thành công 90% giao diện với API thật, hệ thống sẵn sàng cho giai đoạn kiểm thử tổng thể.
