---
title: "Worklog Tuần 4"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4

- Mở rộng kiến thức chuyên sâu về Terraform và Docker containerization cho ứng dụng full-stack
- Tự học GitHub Actions CI/CD và triển khai quy trình tự động hóa vào repository của nhóm
- Nghiên cứu chuẩn thiết kế sơ đồ kiến trúc AWS và hoàn thiện bản vẽ hạ tầng 3 tầng cho dự án

### Các công việc cần triển khai trong tuần này

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | -------------- |
| 2   | - Nâng cao kiến thức Terraform: thiết kế module tái sử dụng, quản lý file state và cấu hình biến môi trường <br> - Tự học chuyên sâu Docker: multi-stage builds và tối ưu `docker-compose.yml` cho mô hình multi-container | 06/07/2026 | 06/07/2026 | [Terraform Beginner to Pro](https://courses.devopsdirective.com/terraform-beginner-to-pro/lessons), [Docker Beginner to Pro](https://courses.devopsdirective.com/docker-beginner-to-pro/lessons) |
| 3   | - Thực hiện containerize ứng dụng LMS của nhóm: viết `Dockerfile` cho frontend và backend <br> - Cấu hình `docker-compose.yml` chạy đồng thời Node.js backend, Redis cache và cơ sở dữ liệu MySQL ở môi trường local <br> - Kiểm thử môi trường container local đảm bảo tính đồng nhất giữa các máy tính trong nhóm | 07/07/2026 | 07/07/2026 | [Docker Beginner to Pro](https://courses.devopsdirective.com/docker-beginner-to-pro/lessons) |
| 4   | - Tự học GitHub Actions CI/CD: workflows, jobs, steps, runners, sự kiện kích hoạt (`push`, `pull_request`) và quản lý repository secrets <br> - Viết script `.github/workflows/ci.yml` kiểm tra cú pháp code và automated build | 08/07/2026 | 08/07/2026 | [GitHub Actions Beginner to Pro](https://courses.devopsdirective.com/github-actions-beginner-to-pro/lessons) |
| 5   | - Triển khai quy trình GitHub Actions CI/CD vào dự án của nhóm: <br> &nbsp;&nbsp;• Pipeline Frontend: tự động hóa quá trình build và kiểm tra release <br> &nbsp;&nbsp;• Pipeline Backend: tự động kiểm tra code, chạy test và chuẩn bị deployment <br> - Cấu hình an toàn GitHub Repository Secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, v.v.) | 09/07/2026 | 09/07/2026 | [GitHub Actions Beginner to Pro](https://courses.devopsdirective.com/github-actions-beginner-to-pro/lessons) |
| 6   | - Tìm hiểu chuẩn vẽ sơ đồ kiến trúc AWS: biểu tượng AWS Architecture Icons, phân tầng subnet và luồng dữ liệu <br> - Phác thảo sơ đồ kiến trúc AWS 3-tier cho dự án LMS (VPC, public/private subnets, ALB, ASG, RDS, Redis, DynamoDB, S3) | 10/07/2026 | 10/07/2026 | [AWS Architecture Icons](https://aws.amazon.com/architecture/icons/) |
| 7   | - Tích hợp bước kiểm tra Terraform tự động vào GitHub Actions (`terraform fmt`, `terraform validate`, `terraform plan` trên Pull Requests) <br> - Thảo luận sơ đồ kiến trúc với các thành viên trong nhóm, tinh chỉnh vị trí tài nguyên và ranh giới bảo mật | 11/07/2026 | 11/07/2026 | |
| CN  | - Viết tài liệu hướng dẫn thiết lập môi trường Docker và quy trình GitHub Actions CI/CD trong README của nhóm <br> - Kiểm tra hoạt động thực tế của GitHub Actions khi push code lên repository <br> - Tổng hợp kết quả tuần 4 và chuẩn bị mục tiêu cho Tuần 5 | 12/07/2026 | 12/07/2026 | |

### Kết quả đạt được sau tuần 4

- Làm chủ kỹ thuật thiết kế Terraform modular và tối ưu Docker multi-stage containerization
- Containerize thành công ứng dụng LMS của nhóm trên môi trường local bằng Docker và Docker Compose
- Xây dựng và tích hợp thành công hệ thống GitHub Actions CI/CD tự động hóa cho cả Frontend và Backend trong repository dự án
- Cấu hình an toàn Repository Secrets và quy trình kiểm tra Terraform tự động trên GitHub Actions
- Thao tác thành thạo chuẩn vẽ sơ đồ kiến trúc AWS và hoàn thành sơ đồ hạ tầng AWS 3-tier chính thức cho dự án
