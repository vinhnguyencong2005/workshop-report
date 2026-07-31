---
title: "Worklog Week 5"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives

- Complete 3-Tier Enterprise Cloud Architecture design for the LMS platform.
- Standardize API Contracts (OpenAPI/Swagger), Database Schemas (RDS MySQL, Redis, DynamoDB), and security standards.
- Architect storage strategy (S3 Presigned URLs) and plan Terraform IaC / CI-CD automation.
- Track team progress and support Frontend and Backend developers in completing core application features (Auth, Courses, Quizzes, Judge0) in dev environment.

### Tasks Completed in Week 5

| Day | Task | Start Date | End Date | Resources |
| --- | ---- | ---------- | -------- | --------- |
| Mon | **System Architecture & API Contract Specification**: <br> - Designed overall 3-tier AWS architecture (Amplify Frontend + ALB/ASG Backend + RDS/Redis/DynamoDB) <br> - Built OpenAPI/Swagger documentation defining request/response JSON schemas for Frontend & Backend <br> - *Team Progress*: Backend built Auth API module; Frontend built application shell UI | 13/07/2026 | 13/07/2026 | AWS Architecture Framework |
| Tue | **Database & Storage Architecture Design**: <br> - Designed relational database schema (RDS MySQL Multi-AZ) and DynamoDB audit tables <br> - Designed ElastiCache Redis caching layer strategy for session management and API rate-limiting <br> - Architected direct S3 file upload flow using S3 Presigned URLs (preventing EC2 memory overload) <br> - *Team Progress*: Backend wrote ORM models & controllers; Frontend built Course management views | 14/07/2026 | 14/07/2026 | AWS RDS & S3 Best Practices |
| Wed | **Networking & Security Baseline Design**: <br> - Planned VPC network layout (Public/Private subnets), Security Groups, and IAM Roles <br> - Designed Regional AWS WAF v2 protection against OWASP Top 10 exploits and API Gateway HTTP API HTTPS proxy <br> - Standardized CORS headers and SSL/TLS to prevent Mixed Content security browser blocks <br> - *Team Progress*: Backend integrated Quiz & Judge0 code grading APIs; Frontend connected Course & Lecture views | 15/07/2026 | 15/07/2026 | AWS Security Baseline |
| Thu | **Terraform IaC Planning & Deployment Strategy**: <br> - Planned modular Terraform architecture (`vpc.tf`, `alb.tf`, `ec2.tf`, `amplify.tf`, `waf.tf`) <br> - Drafted EC2 bootstrap script (`user_data.tftpl`) and immutable deployment model <br> - Planned GitHub Actions CI/CD workflows for Frontend (Amplify) and Backend (ASG Rolling Refresh) <br> - *Team Progress*: Backend finalized remaining CRUD APIs; Frontend completed Quiz UI & Code Submission views | 16/07/2026 | 16/07/2026 | Terraform Best Practices |
| Fri | **Internal Integration Testing & Developer Support**: <br> - Supported Frontend & Backend developers in connecting APIs, resolving CORS, Header, and `.env` blockers <br> - Conducted joint End-to-End integration testing on local dev environment <br> - Reviewed PR code, optimized DB queries, and standardized Postman API collection for the team <br> - Mid-project sync: Development team completed 100% application features, ready for Week 6 cloud provisioning | 17/07/2026 | 17/07/2026 | Dev Integration Guidelines |

### Week 5 Achievements

- **Design & Specification**: 100% completed 3-tier architecture documentation, DB ERD diagrams, and standardized API Contract specifications.
- **Developer Enablement**:
  - Clear API Contracts enabled parallel Frontend & Backend development without integration bottlenecks.
  - Resolved local CORS and API connection issues for dev team.
- **Team Progress**:
  - **Backend**: 100% completed core application APIs (Auth, Courses, Forum, Materials, Quizzes, Judge0 Code Execution).
  - **Frontend**: Successfully connected UI components with backend APIs on local dev environment, ready for AWS deployment in Week 6.
