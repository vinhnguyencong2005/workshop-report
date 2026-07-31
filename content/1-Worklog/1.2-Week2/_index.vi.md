---
title: "Worklog Tuần 2"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2

- Khám phá containerization với Docker và container orchestration trên AWS (ECS)
- Làm quen với khái niệm Infrastructure as Code (IaC) thông qua Terraform và AWS CloudFormation
- Tiếp tục hoàn thành các bài lab thực hành AWS từ chương trình study group

### Các công việc cần triển khai trong tuần này

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | -------------- |
| 2   | - Tìm hiểu Docker căn bản: images, containers, Dockerfile, docker-compose <br> - Cài đặt Docker Desktop và chạy ứng dụng containerized đầu tiên trên máy local <br> - Phân biệt kiến trúc container so với virtual machine | 22/06/2026 | 22/06/2026 | [Docker Beginner to Pro](https://courses.devopsdirective.com/docker-beginner-to-pro/lessons) |
| 3   | - Lab: **Deploy Application on Docker** — build Docker image, chạy containers, quản lý container lifecycle <br> - Lab: **Deploy applications on Amazon ECS** — tạo ECS cluster, task definitions và services <br> - Tìm hiểu mối quan hệ giữa Docker containers và AWS ECS/Fargate | 23/06/2026 | 23/06/2026 | [000015.awsstudygroup.com](https://000015.awsstudygroup.com/), [000016.awsstudygroup.com](https://000016.awsstudygroup.com/) |
| 4   | - Lab: **Deploying CI/CD with ECS Container** — thiết lập CI/CD pipeline cho containerized applications trên ECS <br> - Lab: **AWS CloudFormation** — tìm hiểu khái niệm IaC, viết CloudFormation templates, deploy stacks <br> - So sánh CloudFormation (AWS-native) vs Terraform (multi-cloud) | 24/06/2026 | 24/06/2026 | [000017.awsstudygroup.com](https://000017.awsstudygroup.com/), [000037.awsstudygroup.com](https://000037.awsstudygroup.com/) |
| 5   | - Làm quen Terraform: HCL syntax, providers, resources, state files <br> - Cài đặt Terraform CLI, viết file `main.tf` đầu tiên để provision S3 bucket <br> - Nắm vững Terraform workflow: `init` → `plan` → `apply` → `destroy` <br> - Lab: **Deploy AWS Backup to the System** — cấu hình automated backup strategies | 25/06/2026 | 25/06/2026 | [Terraform Beginner to Pro](https://courses.devopsdirective.com/terraform-beginner-to-pro/lessons), [000013.awsstudygroup.com](https://000013.awsstudygroup.com/) |
| 6   | - Lab: **Setting up VPC Peering** — kết nối nhiều VPC và cấu hình routing giữa các VPC <br> - Lab: **Set up AWS Transit Gateway** — centralized networking hub cho multi-VPC architectures <br> - Phân biệt các mô hình networking nâng cao: VPC Peering vs Transit Gateway | 26/06/2026 | 26/06/2026 | [000019.awsstudygroup.com](https://000019.awsstudygroup.com/), [000020.awsstudygroup.com](https://000020.awsstudygroup.com/) |
| 7   | - Lab: **Getting Started with AWS Security Hub** — kích hoạt tổng hợp security findings và compliance checks <br> - Lab: **AWS Web Application Firewall** — cấu hình WAF rules bảo vệ web applications <br> - Lab: **Managing Resources with Tags** — tổ chức và quản lý tài nguyên AWS bằng tagging strategies | 27/06/2026 | 27/06/2026 | [000018.awsstudygroup.com](https://000018.awsstudygroup.com/), [000026.awsstudygroup.com](https://000026.awsstudygroup.com/), [000027.awsstudygroup.com](https://000027.awsstudygroup.com/) |
| CN  | - Thực hành Terraform: viết modules cho VPC và EC2, tìm hiểu variables và outputs <br> - Ôn tập và tổng hợp kiến thức Docker, Terraform, và CloudFormation <br> - So sánh các công cụ IaC và quyết định chọn Terraform cho dự án nhóm <br> - Lên kế hoạch cho tuần kế tiếp | 28/06/2026 | 28/06/2026 | [Terraform Beginner to Pro](https://courses.devopsdirective.com/terraform-beginner-to-pro/lessons) |

### Kết quả đạt được sau tuần 2

- Nắm vững khái niệm Docker containerization: images, containers, Dockerfile, và container lifecycle management
- Hoàn thành các bài lab liên quan đến container trên AWS: Docker deployment, ECS cluster provisioning, và CI/CD pipeline cho containers
- Xây dựng kiến thức nền tảng về Infrastructure as Code (IaC) với cả AWS CloudFormation và Terraform
- Viết được các cấu hình Terraform đầu tiên: S3 bucket, VPC module, và EC2 instance sử dụng HCL syntax
- Khám phá advanced AWS networking: VPC Peering, Transit Gateway, và multi-VPC routing architectures
- Hoàn thành các bài lab về bảo mật: AWS Security Hub, WAF, và resource tagging best practices
- Quyết định sử dụng Terraform làm công cụ IaC chính cho dự án nhóm nhờ tính linh hoạt multi-cloud và thiết kế modular
