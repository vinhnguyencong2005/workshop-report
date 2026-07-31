---
title: "Worklog Tuần 5"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5

- Hoàn thiện bản thiết kế kiến trúc hệ thống cloud 3 tầng (3-Tier AWS Architecture) cho toàn bộ dự án LMS.
- Định nghĩa API Contract (OpenAPI/Swagger), chuẩn hóa mô hình dữ liệu (RDS MySQL, Redis, DynamoDB) và quy chuẩn bảo mật.
- Thiết kế chiến lược lưu trữ (S3 Presigned URL) và lập kế hoạch triển khai mã hạ tầng Terraform / CI-CD pipeline.
- Theo dõi tiến độ và hỗ trợ các lập trình viên Frontend và Backend hoàn thiện các chức năng cốt lõi (Auth, Courses, Quizzes, Judge0) trên môi trường dev.

### Các công việc triển khai trong tuần 5

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | -------------- |
| 2   | **Thiết kế Kiến trúc Hệ thống & Quy chuẩn API Contract**: <br> - Thiết kế sơ đồ kiến trúc tổng thể 3 tầng trên AWS (Amplify Frontend + ALB/ASG Backend + RDS/Redis/DynamoDB) <br> - Xây dựng tài liệu OpenAPI/Swagger định nghĩa chuẩn dữ liệu request/response giữa Frontend & Backend <br> - *Tiến độ nhóm*: Backend dựng module Auth API; Frontend xây dựng UI layout khung ứng dụng | 13/07/2026 | 13/07/2026 | AWS Architecture Framework |
| 3   | **Thiết kế Cơ sở Dữ liệu & Giải pháp Lưu trữ**: <br> - Thiết kế mô hình cơ sở dữ liệu quan hệ RDS MySQL (Multi-AZ) và các bảng DynamoDB audit log <br> - Thiết kế chiến lược bộ nhớ đệm ElastiCache Redis cho quản lý session và rate-limiting <br> - Thiết kế giải pháp upload file tài liệu trực tiếp lên S3 qua S3 Presigned URL (tránh quá tải EC2) <br> - *Tiến độ nhóm*: Backend viết ORM models & controllers; Frontend dựng trang quản lý khóa học | 14/07/2026 | 14/07/2026 | AWS RDS & S3 Best Practices |
| 4   | **Quy hoạch Mạng & Thiết kế Mô hình Bảo mật**: <br> - Quy hoạch cấu trúc VPC (Public/Private subnets), Security Groups và phân quyền IAM Roles <br> - Đề xuất mô hình Regional AWS WAF v2 chống OWASP Top 10 và API Gateway HTTP API proxy HTTPS <br> - Thiết kế giải pháp chuẩn hóa CORS headers và SSL/TLS để ngăn ngừa lỗi Mixed Content <br> - *Tiến độ nhóm*: Backend tích hợp API Quiz & Judge0 chấm code; Frontend làm UI bài giảng & bài tập | 15/07/2026 | 15/07/2026 | AWS Security Baseline |
| 5   | **Kế hoạch Hạ tầng Terraform & Chiến lược Deploy**: <br> - Lập sơ đồ phân rã các module Terraform (`vpc.tf`, `alb.tf`, `ec2.tf`, `amplify.tf`, `waf.tf`) <br> - Phác thảo cấu trúc script bootstrap EC2 (`user_data.tftpl`) và mô hình deploy bất biến (Immutable) <br> - Lập kế hoạch thiết lập GitHub Actions CI/CD cho Frontend (Amplify) và Backend (ASG Rolling Refresh) <br> - *Tiến độ nhóm*: Backend hoàn thiện các CRUD API còn lại; Frontend hoàn thành UI làm bài Quiz & nộp code | 16/07/2026 | 16/07/2026 | Terraform Best Practices |
| 6   | **Kiểm thử Tích hợp Nội bộ & Hỗ trợ Đội ngũ Phát triển**: <br> - Hỗ trợ Frontend & Backend ghép nối API, sửa các lỗi vướng mắc về CORS, Headers và biến môi trường `.env` <br> - Tổ chức phiên kiểm thử tích hợp ứng dụng (End-to-End integration test) trên môi trường phát triển cục bộ <br> - Review PR code, tối ưu hóa câu lệnh truy vấn CSDL và chuẩn hóa Postman API collection cho nhóm <br> - Họp tổng kết tiến độ tuần 5: Đội ngũ phát triển hoàn thiện 100% tính năng ứng dụng, sẵn sàng cho tuần 6 triển khai cloud | 17/07/2026 | 17/07/2026 | Dev Integration Guidelines |

### Kết quả đạt được sau tuần 5

- **Thiết kế & Quy chuẩn**: Hoàn thành 100% tài liệu thiết kế kiến trúc 3 tầng, sơ đồ ERD cơ sở dữ liệu và bộ tài liệu API Contract chuẩn hóa.
- **Hỗ trợ Đội ngũ Phát triển**:
  - API Contract rõ ràng giúp Frontend và Backend dev triển khai song song không bị nghẽn (unblock).
  - Giải quyết các vấn đề vướng mắc về CORS và tích hợp API cục bộ cho nhóm.
- **Tiến độ Dự án**:
  - **Backend**: Hoàn thành 100% các API cốt lõi (Auth, Courses, Forum, Materials, Quizzes, Judge0 Code Execution).
  - **Frontend**: Ghép nối thành công giao diện với các API backend trên môi trường dev, chuẩn hóa mã nguồn cho bước triển khai hạ tầng AWS ở tuần 6.
