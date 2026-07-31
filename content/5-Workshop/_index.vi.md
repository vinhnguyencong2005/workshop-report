---
title: "Workshop"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5. </b> "
includeInReport: false
---

# Triển khai LMS Full-Stack trên AWS với Terraform

#### Tổng quan

Trong workshop này, ta sẽ triển khai một hệ thống **Learning Management System (LMS) 3 tầng** hoàn chỉnh trên AWS bằng **Terraform** — Infrastructure as Code (IaC). Ta sẽ khởi tạo VPC với public và private subnets, AWS WAF v2, AWS Amplify Hosting, API Gateway HTTP API, EC2 Auto Scaling Group chạy Node.js, RDS MySQL Multi-AZ, ElastiCache Redis, DynamoDB, và S3 — hoàn toàn bằng code `.tf`. Không thao tác thủ công trên AWS Console. Cuối bài lab, lệnh `terraform destroy` sẽ tự động dọn dẹp toàn bộ tài nguyên.

#### Nội dung

1. [Tổng quan về Workshop](5.1-Workshop-overview/)
2. [Các bước chuẩn bị](5.2-Prerequiste/)
3. [Mạng & Bảo mật](5.3-networking-security/)
4. [Tầng dữ liệu](5.4-data-layer/)
5. [Tầng tính toán & Cân bằng tải](5.5-compute-alb/)
6. [Giám sát hệ thống](5.6-monitoring/)
7. [Triển khai & Kiểm tra](5.7-deploy-verify/)
8. [CI/CD](5.8-cicd/)
9. [Dọn dẹp tài nguyên](5.9-cleanup/)