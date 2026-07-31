---
title: "Worklog Tuần 4"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4

- Tự học mở rộng về các module Terraform và kiến thức Docker containerization
- Học GitHub Actions CI/CD và trực tiếp triển khai quy trình tự động hóa vào repository của nhóm
- Nghiên cứu chuẩn thiết kế sơ đồ kiến trúc AWS và hoàn thiện bản vẽ hạ tầng 3 tầng cho dự án

### Các công việc cần triển khai trong tuần này

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | -------------- |
| 2   | - Nâng cao kiến thức Terraform: thiết kế module tái sử dụng, quản lý file state và cấu hình biến môi trường <br> - Tự học chuyên sâu Docker: multi-stage builds và các mô hình cấu hình container | 06/07/2026 | 06/07/2026 | [Terraform Beginner to Pro](https://courses.devopsdirective.com/terraform-beginner-to-pro/lessons), [Docker Beginner to Pro](https://courses.devopsdirective.com/docker-beginner-to-pro/lessons) |
| 3   | - Thực hành Docker containerization trên môi trường local: thử nghiệm viết `Dockerfile` và chạy container <br> - Đánh giá việc đưa Docker vào dự án LMS (xác định không đưa Docker vào sản phẩm thực tế để giữ quy trình triển khai đơn giản, tinh gọn) | 07/07/2026 | 07/07/2026 | [Docker Beginner to Pro](https://courses.devopsdirective.com/docker-beginner-to-pro/lessons) |
| 4   | - Tự học GitHub Actions CI/CD: workflows, jobs, steps, runners, sự kiện kích hoạt (`push`, `pull_request`) và quản lý repository secrets <br> - Viết script `.github/workflows/ci.yml` kiểm tra cú pháp code và automated build | 08/07/2026 | 08/07/2026 | [GitHub Actions Beginner to Pro](https://courses.devopsdirective.com/github-actions-beginner-to-pro/lessons) |
| 5   | - Triển khai quy trình GitHub Actions CI/CD vào dự án của nhóm: <br> &nbsp;&nbsp;• Pipeline Frontend: tự động hóa quá trình build và kiểm tra release <br> &nbsp;&nbsp;• Pipeline Backend: tự động kiểm tra code, chạy test và kích hoạt deployment <br> - Cấu hình an toàn GitHub Repository Secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, v.v.) | 09/07/2026 | 09/07/2026 | [GitHub Actions Beginner to Pro](https://courses.devopsdirective.com/github-actions-beginner-to-pro/lessons) |
| 6   | - Tìm hiểu chuẩn vẽ sơ đồ kiến trúc AWS: biểu tượng AWS Architecture Icons, phân tầng subnet và luồng dữ liệu <br> - Phác thảo sơ đồ kiến trúc AWS 3-tier cho dự án LMS (VPC, public/private subnets, ALB, ASG, RDS, Redis, DynamoDB, S3) | 10/07/2026 | 10/07/2026 | [AWS Architecture Icons](https://aws.amazon.com/architecture/icons/) |
| 7   | - Tích hợp bước kiểm tra Terraform tự động vào GitHub Actions (`terraform fmt`, `terraform validate`, `terraform plan` trên Pull Requests) <br> - Thảo luận sơ đồ kiến trúc với các thành viên trong nhóm, tinh chỉnh vị trí tài nguyên và ranh giới bảo mật | 11/07/2026 | 11/07/2026 | |
| CN  | - Viết tài liệu hướng dẫn quy trình GitHub Actions CI/CD trong repository của nhóm <br> - Kiểm tra hoạt động thực tế của GitHub Actions khi push code lên repository <br> - Tổng hợp kết quả tuần 4 và chuẩn bị mục tiêu cho Tuần 5 | 12/07/2026 | 12/07/2026 | |

### Kết quả đạt được sau tuần 4

- Củng cố kiến thức về thiết kế Terraform modular và nguyên lý hoạt động của Docker containerization
- Đánh giá thực tế Docker trên môi trường local (thống nhất giữ quy trình triển khai ứng dụng trực tiếp trên EC2 cho sản phẩm của nhóm)
- Triển khai thành công hệ thống GitHub Actions CI/CD tự động hóa cho cả Frontend và Backend trong repository dự án
- Cấu hình an toàn Repository Secrets và quy trình kiểm tra Terraform tự động trên GitHub Actions
- Thao tác thành thạo chuẩn vẽ sơ đồ kiến trúc AWS và hoàn thành sơ đồ hạ tầng AWS 3-tier chính thức cho dự án
