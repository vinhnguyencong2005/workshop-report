---
title: "Worklog Week 1"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Week 1 Objectives

- Find, get acquainted with, and form a team in the FCAJ program
- Get familiar with AWS cloud fundamentals and core services
- Complete AWS hands-on lab exercises covering compute, networking, storage, database, monitoring, and edge services

### Tasks to Complete This Week

| Day | Task | Start Date | End Date | Resources |
| --- | ---- | ---------- | -------- | --------- |
| Mon | - Find and form a team with FCAJ members <br> - Get acquainted with team members <br> - Read and note the rules and regulations at the internship site <br> - Learn what "the Cloud" is and its infrastructure <br> - AWS management tools (AWS services) <br> - Videos: Module 01-01 → Module 01-05 | June 15, 2026 | June 15, 2026 | [First Cloud Journey Kick off 2024](https://www.youtube.com/watch?v=AQlsd0nWdZk&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=1) |
| Tue | - Hands-on: Create an AWS account <br> - Hands-on: Get $100 credit from AWS <br> - Lab: **Managing Costs with AWS Budgets** — set up budget alerts and cost monitoring <br> - Lab: **Getting Help with AWS Support** — explore support plans and create support cases <br> - Videos: Module 01-Lab01-01 → Module 01-Lab01-04 | June 16, 2026 | June 16, 2026 | [000001.awsstudygroup.com](https://000001.awsstudygroup.com/), [000007.awsstudygroup.com](https://000007.awsstudygroup.com/), [000009.awsstudygroup.com](https://000009.awsstudygroup.com/) |
| Wed | - Lab: **Access Management with AWS IAM** — create IAM users, groups, policies, and MFA <br> - Lab: **Networking Essentials with Amazon VPC** — build VPC, subnets, route tables, Internet Gateway <br> - Lab: **Compute Essentials with Amazon EC2** — launch instances, configure security groups, SSH access <br> - Lab: **Instance Profiling with IAM Roles for EC2** — attach IAM roles to EC2, access AWS services without credentials | June 17, 2026 | June 17, 2026 | [000002.awsstudygroup.com](https://000002.awsstudygroup.com/), [000003.awsstudygroup.com](https://000003.awsstudygroup.com/), [000004.awsstudygroup.com](https://000004.awsstudygroup.com/), [000048.awsstudygroup.com](https://000048.awsstudygroup.com/) |
| Thu | - Lab: **Cloud Development with AWS Cloud9** — set up cloud IDE environment for development <br> - Lab: **Static Website Hosting with Amazon S3** — create S3 bucket, enable static website, configure bucket policy <br> - Lab: **Database Essentials with Amazon RDS** — provision MySQL/PostgreSQL instance, connect from EC2 <br> - Lab: **Command Line Operations with AWS CLI** — install, configure, and manage AWS resources via CLI | June 18, 2026 | June 18, 2026 | [000049.awsstudygroup.com](https://000049.awsstudygroup.com/), [000057.awsstudygroup.com](https://000057.awsstudygroup.com/), [000005.awsstudygroup.com](https://000005.awsstudygroup.com/), [000011.awsstudygroup.com](https://000011.awsstudygroup.com/) |
| Fri | - Lab: **Scaling Applications with EC2 Auto Scaling** — create launch template, ASG, scaling policies <br> - Lab: **Monitoring with Amazon CloudWatch** — set up dashboards, alarms, and log groups <br> - Lab: **NoSQL Database Essentials with Amazon DynamoDB** — create tables, CRUD operations, secondary indexes <br> - Lab: **In-Memory Caching with Amazon ElastiCache** — provision Redis cluster, test caching from EC2 | June 19, 2026 | June 19, 2026 | [000006.awsstudygroup.com](https://000006.awsstudygroup.com/), [000008.awsstudygroup.com](https://000008.awsstudygroup.com/), [000060.awsstudygroup.com](https://000060.awsstudygroup.com/), [000061.awsstudygroup.com](https://000061.awsstudygroup.com/) |
| Sat | - Lab: **Hybrid DNS Management with Amazon Route 53** — register hosted zones, configure DNS records <br> - Lab: **Networking on AWS Workshop** — advanced VPC peering, transit gateway, VPN concepts <br> - Lab: **Content Delivery with Amazon CloudFront** — create CloudFront distribution, configure origins and cache behaviors <br> - Lab: **Edge Computing with CloudFront and Lambda@Edge** — deploy Lambda functions at edge locations for request/response manipulation | June 20, 2026 | June 20, 2026 | [000010.awsstudygroup.com](https://000010.awsstudygroup.com/), [000092.awsstudygroup.com](https://000092.awsstudygroup.com/), [000094.awsstudygroup.com](https://000094.awsstudygroup.com/) |
| Sun | - Review & summarize all lab exercises and knowledge learned this week <br> - Take notes on key services: IAM, VPC, EC2, S3, RDS, DynamoDB, ElastiCache, CloudWatch, Route 53, CloudFront <br> - Document architecture patterns observed across labs <br> - Prepare for the upcoming week | June 21, 2026 | June 21, 2026 | |

### Week 1 Results

- Understood what AWS is, how it operates, and how it organizes regions (Regions, Availability Zones)
- Created an AWS account and obtained $100 credit from AWS
- Completed **18 AWS hands-on lab exercises** covering core services:
  - **Identity & Cost**: IAM (users, groups, roles, policies), AWS Budgets, AWS Support
  - **Networking**: VPC (subnets, route tables, Internet Gateway, NAT), Route 53 DNS, advanced VPC workshop
  - **Compute**: EC2 (launch, connect, security groups), IAM Roles for EC2, EC2 Auto Scaling (launch templates, ASG, scaling policies)
  - **Storage & Database**: S3 static website hosting, RDS MySQL, DynamoDB NoSQL, ElastiCache Redis caching
  - **Monitoring & Edge**: CloudWatch (dashboards, alarms, logs), CloudFront CDN, Lambda@Edge for edge computing
  - **Developer Tools**: AWS Cloud9 IDE, AWS CLI command-line operations
- Gained foundational understanding of AWS networking, compute, storage, database, and monitoring — setting the baseline for the Terraform IaC project in later weeks
