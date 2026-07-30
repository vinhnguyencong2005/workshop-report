---
title : "Data Layer"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

#### What We're Building

Four data stores, each chosen for a specific job:

| Store | Type | What It Holds |
|-------|------|---------------|
| **S3 ×3** | Object storage | Frontend static site, user file uploads, deployment artifacts |
| **RDS MySQL** | Relational (Multi-AZ) | Users, courses, enrollments, grades — structured data with relationships |
| **ElastiCache Redis** | In-memory cache | Sessions, rate limiting, fast lookups |
| **DynamoDB ×5** | NoSQL (key-value) | Course content, forum posts, quizzes, schedules, assignments |

Everything sits in private DB subnets except S3 (regional service) and DynamoDB (regional service). VPC Gateway Endpoints keep S3 and DynamoDB traffic off the public internet.

#### Content

1. [S3 Buckets & VPC Endpoint](5.4.1-s3/) — Three buckets, website hosting, encryption, CORS
2. [RDS MySQL](5.4.2-rds/) — Multi-AZ, 20 GB gp3, 7-day backups
3. [ElastiCache Redis](5.4.3-redis/) — 2-node replication, encryption at-rest + in-transit
4. [DynamoDB & VPC Endpoint](5.4.4-dynamodb/) — Five tables with GSIs, on-demand billing

---
