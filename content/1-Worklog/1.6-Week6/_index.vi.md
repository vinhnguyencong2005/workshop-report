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
| 2   | - Chạy `terraform apply` triển khai EC2 trên public subnet <br> - Cài đặt Node.js, PM2, clone source code backend thủ công lên instance <br> - Cấu hình biến môi trường (.env) và kiểm tra backend hoạt động qua public IP | 20/07/2026 | 20/07/2026 | |
| 3   | - Build và deploy frontend: `npm run build` → upload lên S3 static website hosting <br> - Kiểm tra frontend kết nối được đến backend qua public IP <br> - Bắt đầu nghiên cứu chuyển EC2 từ public subnet sang private subnet để tăng bảo mật | 21/07/2026 | 21/07/2026 | |
| 4   | - Refactor Terraform: chuyển EC2 vào private subnet, cấu hình NAT Gateway <br> - Thử nghiệm phương án deploy: upload code lên S3 → gửi lệnh SSM Run Command để instance pull code từ S3 về <br> - Thử đăng ký Cloudflare làm CDN nhưng dịch vụ từ chối tài khoản, quyết định bỏ qua | 22/07/2026 | 22/07/2026 | |
| 5   | - Tối ưu chiến lược deploy: thay thế SSM trigger bằng cơ chế instance tự pull code từ S3 khi boot (user_data) <br> - Viết script `user_data.tftpl`: cài AWS CLI, Node.js, PM2, pull backend.zip, giải nén, chạy app <br> - Cấu hình DynamoDB trên AWS và kiểm tra kết nối từ backend | 23/07/2026 | 23/07/2026 | |
| 6   | - Thiết lập GitHub Actions workflow cho backend: zip source → upload S3 → trigger ASG instance refresh <br> - Build script deploy cho frontend: `aws s3 sync dist/` lên S3 bucket <br> - Viết IAM policy cho EC2 (quyền S3, DynamoDB, Secrets Manager, SSM) | 24/07/2026 | 24/07/2026 | |
| 7   | - Kiểm thử toàn bộ luồng deploy tự động: push code → GitHub Actions chạy → ASG refresh → app hoạt động <br> - Ghi nhận và sửa các lỗi phát sinh: CORS, health check timeout, security group rule thiếu <br> - Tinh chỉnh ALB target group health check để giảm thời gian phát hiện instance lỗi | 25/07/2026 | 25/07/2026 | |
| CN  | - Dọn dẹp và tổ chức lại cấu trúc thư mục Terraform <br> - Viết tài liệu mô tả kiến trúc hạ tầng và quy trình deploy <br> - Chạy kiểm thử end-to-end lần cuối: đăng nhập, xem khóa học, làm quiz, nộp bài trên môi trường production | 26/07/2026 | 26/07/2026 | |

### Kết quả đạt được sau tuần 6

- Hạ tầng AWS hoàn chỉnh: EC2 trong private subnet, ALB public, RDS, ElastiCache Redis, DynamoDB
- Chiến lược deploy ổn định: backend deploy qua S3 + ASG instance refresh, frontend qua S3 sync
- CI/CD pipeline với GitHub Actions hoạt động, tự động deploy khi push code lên nhánh `main`
- Từ bỏ Cloudflare sau khi dịch vụ từ chối tài khoản, sử dụng trực tiếp ALB DNS và S3 website endpoint
- Tài liệu kiến trúc hạ tầng và quy trình deploy đã được viết, sẵn sàng cho giai đoạn vận hành
