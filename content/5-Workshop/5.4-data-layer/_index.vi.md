---
title : "Tầng Dữ liệu (Data Layer)"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

#### Các Dịch vụ Dữ liệu

Bốn kho lưu trữ dữ liệu, mỗi kho được lựa chọn cho một nhiệm vụ cụ thể:

| Kho lưu trữ | Loại | Dữ liệu quản lý |
|-------|------|---------------|
| **S3 ×3** | Lưu trữ đối tượng (Object storage) | Bản sao lưu frontend, tệp tải lên của người dùng, artifact triển khai |
| **RDS MySQL** | Cơ sở dữ liệu quan hệ (Multi-AZ) | Người dùng, khóa học, đăng ký, điểm số — dữ liệu có cấu trúc và quan hệ |
| **ElastiCache Redis** | Cache bộ nhớ (In-memory) | Phiên làm việc (sessions), giới hạn tần suất (rate limiting), truy xuất nhanh |
| **DynamoDB ×5** | NoSQL (Key-Value) | Nội dung khóa học, bài viết diễn đàn, bài kiểm tra, lịch trình, bài tập |
| **AWS Amplify** | Hosting & CDN | Lưu trữ và phân phối ứng dụng web tĩnh tĩnh React/Vite |

Tất cả nằm trong private DB subnets ngoại trừ S3 và DynamoDB (các dịch vụ AWS regional). VPC Gateway Endpoints giúp lưu lượng truy cập S3 và DynamoDB không đi qua internet công cộng.

#### Nội dung

1. [S3 Buckets & VPC Endpoint](5.4.1-s3/) — Ba buckets riêng tư, mã hóa, CORS
2. [RDS MySQL](5.4.2-rds/) — Multi-AZ, 20 GB gp3, sao lưu tự động 7 ngày
3. [ElastiCache Redis](5.4.3-redis/) — Nhân bản 2-node, mã hóa at-rest + in-transit
4. [DynamoDB & VPC Endpoint](5.4.4-dynamodb/) — Năm bảng với GSIs, thanh toán theo nhu cầu (on-demand)
5. [AWS Amplify Hosting](5.4.5-amplify/) — Hosting ứng dụng web React/Vite với chứng chỉ SSL tự động và CDN toàn cầu
