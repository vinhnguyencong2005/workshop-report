---
title : "Workshop Overview"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Introduction

In this workshop, we will deploy a **full-stack Learning Management System** on AWS — entirely with **Terraform**. Instead of clicking through the AWS Console, we will define every resource as code: networking, compute, databases, caching, storage, security, and monitoring. At the end, a single `terraform destroy` tears everything down cleanly.

This is a hands-on, step-by-step journey from an empty AWS account to a production-grade, scalable web application — no prior Terraform experience required.

![Architecture Diagram](../../../images/workshop/aws_architecture.png)

A **3-tier architecture** spanning two Availability Zones, where:

- **Tier 1 (Public):** The Application Load Balancer is the single entry point — only port 80 is exposed to the internet.
- **Tier 2 (Private App):** EC2 instances run Node.js with PM2 inside an Auto Scaling Group. No public IPs. Outbound internet goes through NAT Gateways. Inbound traffic comes **only** from the ALB.
- **Tier 3 (Private DB):** RDS MySQL (Multi-AZ) and ElastiCache Redis sit in the deepest, most isolated subnets. Only the app tier can reach them.

AWS services the application **leverages** (fully managed — we don't install or patch them):

| Category | Service | What It Does for the App |
|----------|---------|--------------------------|
| **Compute** | EC2 Auto Scaling | Runs Node.js backend; replaces failed instances automatically |
| **Load Balancing** | Application Load Balancer | Distributes traffic across EC2 instances; health checks |
| **Relational Database** | RDS MySQL (Multi-AZ) | Stores users, courses, enrollments, grades |
| **In-Memory Cache** | ElastiCache Redis | Sessions, rate limiting, fast lookups |
| **NoSQL Database** | DynamoDB | Course content, forum posts, quizzes, schedules |
| **Object Storage** | S3 (3 buckets) | Static frontend hosting, file uploads, deployment artifacts |
| **Management & Governance** | CloudWatch + SSM | Alarms (ALB errors, CPU, scaling); Session Manager (no SSH needed) |

#### Why Terraform?

| Reason | What It Means |
|--------|-----------------------|
| **Declarative** | We write *what* we want; Terraform figures out *how* to create it |
| **Single workflow** | `init → plan → apply → destroy` — four commands for everything |
| **State tracking** | Terraform knows what it created and detects drift |
| **Clean teardown** | `terraform destroy` removes every resource in minutes — no hunting through the console |
| **Transferable skills** | The patterns we learn (variables, outputs, remote state) apply to any cloud project |

#### What is the outcome

By the end of this workshop, we will be able to:

- Create an IAM user and generate AWS access keys
- Install and configure Terraform + AWS CLI
- Design a multi-tier VPC with public and private subnets
- Route traffic with Internet Gateways, NAT Gateways, and VPC Endpoints
- Write Security Groups that follow least-privilege principles
- Assign IAM roles so EC2 instances can access S3, DynamoDB, RDS, and ElastiCache — without hardcoded keys
- Deploy RDS MySQL with Multi-AZ for high availability
- Deploy ElastiCache Redis with encryption at-rest and in-transit
- Design DynamoDB tables with Global Secondary Indexes for flexible queries
- Configure S3 buckets for static website hosting, private storage, and CORS
- Create an Auto Scaling Group with a Launch Template and user data bootstrapping
- Front the application with an Application Load Balancer and health checks
- Monitor the stack with CloudWatch alarms
- Connect to private EC2 instances securely via SSM Session Manager
- Configure GitHub Actions with secrets for automated CI/CD deployment
- *(Optional)* Set up a Cloudflare custom domain and point it to ALB and S3
- Destroy every resource with a single command

#### Architecture Principles

{{% notice info %}}
Throughout the workshop, the architecture follows these principles:

- **Least privilege:** Every security group rule and IAM policy grants *only* what's needed.
- **High availability:** Two Availability Zones for ALB, EC2, RDS, and Redis. If one AZ fails, the app keeps running.
- **No public IPs on app/db:** EC2, RDS, and Redis sit in private subnets. Only the ALB faces the internet.
- **Encryption everywhere:** RDS, Redis, and S3 are all encrypted. IMDSv2 required on EC2.
- **No SSH:** SSM Session Manager provides shell access without opening port 22.
{{% /notice %}}

#### Estimated Time

| Phase | Activity | Time |
|-------|----------|------|
| Setup | Install Terraform, AWS CLI, Git; create IAM user + access keys; `aws configure`; `terraform init` | 15 min |
| Networking | VPC (CIDR, DNS), 6 subnets (2 public + 4 private), IGW, NAT GW ×2, route tables, VPC endpoints (Gateway + Interface), 5 security groups with rules, IAM role + inline policy + instance profile | 15 min |
| Data Layer | S3 ×3 (website hosting, encryption, versioning, CORS, bucket policy), RDS MySQL (subnet group, Multi-AZ, encryption, backups), ElastiCache Redis (subnet group, 2-node replication, encryption), DynamoDB ×5 (PK/SK design, GSIs, PITR) | 15 min |
| Compute | Launch template (AMI, user data bootstrap, IMDSv2, EBS), Auto Scaling Group (min/desired/max, ELB health checks, private subnets), ALB (internet-facing, target group :3000, HTTP listener, health check path) | 15 min |
| Monitoring | SNS topic, CloudWatch alarms (ALB 5xx, RDS CPU >80%, ASG below min) | 10 min |
| Deploy & Verify | `terraform apply`, SSM Session Manager, curl health endpoint, seed data, test API | 15 min |
| CI/CD | GitHub Actions secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `VITE_API_BASE_URL`), push code, deploy frontend to S3, deploy backend to EC2 | 10 min |
| Custom Domain *(optional)* | Cloudflare domain + nameservers, wire ALB DNS to `api.<domain>`, wire S3 to `app.<domain>` | 5 min |
| Clean Up | `terraform destroy`, verify all resources removed | 10 min |
| **Total** | | **~1.75 hours** (base) / **~2 hours** (with optional)

#### Content

1. [Workshop Overview](5.1-Workshop-overview/) ← We are here
2. [Prerequisites](5.2-Prerequiste/)
3. [Networking & Security](5.3-networking-security/)
4. [Data Layer](5.4-data-layer/)
5. [Compute & Load Balancing](5.5-compute-alb/)
6. [Monitoring](5.6-monitoring/)
7. [Deploy & Verify](5.7-deploy-verify/)
8. [CI/CD](5.8-cicd/)
9. [Custom Domain *(optional)*](5.9-custom-domain/)
10. [Clean Up](5.10-cleanup/)