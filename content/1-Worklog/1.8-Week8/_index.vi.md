---
title: "Worklog Tuần 8"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8

- Tự nghiên cứu & nâng cao kiến thức về Bảo mật Cloud chuyên sâu (AWS IAM, WAF v2, KMS Encryption).
- Tự học kỹ thuật Tối ưu hóa Hiệu năng hệ thống (MySQL Indexing, Redis Cache-Aside Pattern, EC2 Auto-scaling).
- Nghiên cứu các quy chuẩn CI/CD và quy trình triển khai không gián đoạn (Zero-Downtime Deployment).

### Các công việc tự học trong tuần 8

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | -------------- |
| 2   | **Tự học Bảo mật IAM & Mã hóa Dữ liệu (AWS KMS)**: <br> - Nghiên cứu nguyên tắc phân quyền tối thiểu (Least Privilege) trong IAM Roles & Policies <br> - Học cấu hình mã hóa dữ liệu lưu trữ (Data at Rest) bằng AWS Key Management Service (KMS) cho S3 và RDS | 03/08/2026 | 03/08/2026 | AWS IAM & KMS Documentation |
| 3   | **Tự học Tối ưu hóa Cơ sở Dữ liệu Quan hệ (RDS MySQL)**: <br> - Nghiên cứu kỹ thuật đánh chỉ mục (Database Indexing) và phân tích truy vấn qua `EXPLAIN` <br> - Học cách tối ưu kích thước Connection Pool trong Node.js/Sequelize để tránh cạn kệt kết nối DB | 04/08/2026 | 04/08/2026 | High Performance MySQL Guide |
| 4   | **Tự học Chiến lược Bộ nhớ đệm (ElastiCache Redis & HTTP Caching)**: <br> - Nghiên cứu các mô hình đệm Cache-Aside và Write-Through với ElastiCache Redis <br> - Tìm hiểu cơ chế HTTP Caching (Cache-Control, ETag headers) để giảm tải cho backend API | 05/08/2026 | 05/08/2026 | Redis Documentation & Web Dev Guides |
| 5   | **Tự nghiên cứu AWS WAF v2 & Phòng chống Tấn công DDoS**: <br> - Nghiên cứu chuyên sâu các bộ luật AWS Managed Rules (OWASP Top 10, IP Reputation, SQLi) <br> - Học cách viết Custom WAF Rules và chiến lược giới hạn tần suất (Rate Limiting) phòng chống botnet | 06/08/2026 | 06/08/2026 | AWS WAF Developer Guide |
| 6   | **Tự học Tối ưu Pipeline CI/CD & Triển khai Bất biến (Immutable Deployment)**: <br> - Nghiên cứu cơ chế Caching trong GitHub Actions để tăng tốc độ build Frontend & Backend <br> - Tìm hiểu mô hình Blue/Green Deployment và Rolling Instance Refresh cho Auto Scaling Group | 07/08/2026 | 07/08/2026 | GitHub Actions & ASG Docs |

### Kết quả đạt được sau tuần 8

- Củng cố kiến thức nền tảng về bảo mật điện toán đám mây (IAM, WAF, KMS Encryption).
- Nắm vững các kỹ thuật tối ưu hóa hiệu năng CSDL MySQL và chiến lược caching với Redis.
- Hiểu rõ nguyên lý thiết kế CI/CD pipeline tối ưu và mô hình triển khai không gián đoạn cho ứng dụng web.
