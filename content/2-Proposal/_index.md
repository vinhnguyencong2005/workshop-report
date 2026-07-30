---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
includeInReport: true
---

# Enterprise 3-Tier LMS Infrastructure on AWS with Terraform
## Automated, Secure, and Scalable Cloud Hosting & CI/CD Pipeline

### 1. Executive Summary
This proposal outlines the design and automated deployment of a production-grade 3-Tier cloud infrastructure on Amazon Web Services (AWS) for the **Learning Management System (LMS)**. Using **Terraform Infrastructure as Code (IaC)**, the platform provisions a highly available, secure, and scalable environment hosting a React/Vite single-page application (SPA) and a Node.js/Express backend API serving university students and instructors.

The solution integrates **AWS Amplify Hosting** for global frontend delivery, **AWS API Gateway HTTP API** for managed HTTPS reverse proxying, **Regional AWS WAF v2** for web application security, an **Application Load Balancer (ALB)** with an **EC2 Auto Scaling Group (ASG)** in private subnets, **RDS MySQL (Multi-AZ)**, **ElastiCache Redis**, **DynamoDB**, and automated **GitHub Actions CI/CD pipelines**.

---

### 2. Problem Statement

#### Existing Challenges
1. **Manual & Unrepeatable Provisioning**: Setting up cloud servers manually via the AWS Management Console leads to configuration drift, human errors, and slow environment duplication.
2. **Security Vulnerabilities**: Hosting EC2 instances in public subnets with SSH port 22 exposed creates high security risks. Direct HTTP connections also trigger browser **Mixed Content** security blocks when called from HTTPS frontends.
3. **Single Point of Failure & Manual Scalability**: Monolithic single-instance setups suffer from downtime during traffic spikes or hardware failures.
4. **Downtime During Updates**: Manual code updates require stopping servers, leading to service interruptions for students taking exams or viewing materials.

#### Proposed Solution
We propose an automated **Infrastructure as Code (IaC)** solution built with **Terraform**:
- **Zero Public Exposure for Compute**: EC2 instances reside exclusively in private subnets with no public IPs. Management access is secured via **AWS SSM Session Manager** (no SSH key management or open port 22).
- **Managed Edge & Gateway Security**: **AWS WAF v2 (Regional)** protects the ALB against OWASP Top 10 exploits, bad inputs, and rate-limiting DDoS attacks. **AWS API Gateway** provides an automated HTTPS endpoint (`https://<api-id>.execute-api.us-east-1.amazonaws.com`) to eliminate Mixed Content errors.
- **Global SPA Hosting**: **AWS Amplify Hosting** serves the React/Vite frontend via global CDN edge locations with automated SSL management.
- **High Availability & Auto-Scaling**: Multi-AZ RDS MySQL, ElastiCache Redis cluster, and EC2 Auto Scaling Group scale dynamically based on CPU utilization.
- **Automated CI/CD**: GitHub Actions workflows deploy frontend updates to Amplify and execute zero-downtime rolling instance refreshes for the backend.

---

#### Key AWS Services & Terraform Modules
- **Networking & Security**: VPC, Public/Private/Database Subnets across 2 AZs, Internet Gateway, NAT Gateways, Security Groups, IAM Roles (`vpc.tf`, `security_groups.tf`, `iam.tf`).
- **Web Application Firewall**: Regional AWS WAF v2 Web ACL with OWASP Top 10, Bad Inputs, IP Reputation, and Rate Limiting (`waf.tf`).
- **Frontend Hosting**: AWS Amplify Hosting for React/Vite SPA with SPA rewrite rules (`amplify.tf`).
- **API Gateway**: HTTP API for managed HTTPS proxying to ALB (`apigateway.tf`).
- **Compute & Load Balancing**: Application Load Balancer, Launch Template, EC2 Auto Scaling Group with SSM managed instance profile (`alb.tf`, `ec2.tf`).
- **Database & Storage Layer**: RDS MySQL Multi-AZ (`rds.tf`), ElastiCache Redis (`redis.tf`), DynamoDB 5 tables (`dynamodb.tf`), S3 Private Buckets (`s3.tf`) with S3 & DynamoDB VPC Gateway Endpoints.
- **Monitoring & Logging**: CloudWatch Log Groups and CloudWatch Alarm CPU utilization metrics (`cloudwatch.tf`).

---

### 4. Technical Implementation & Timeline

| Phase | Milestone / Tasks | Timeline |
| :--- | :--- | :--- |
| **Phase 1: Architecture & IaC Design** | Research AWS 3-tier architecture, design HCL module structure, define variables (`variables.tf`) | Week 1 - 2 |
| **Phase 2: Core VPC & Security Baseline** | Write Terraform code for VPC, multi-AZ subnets, NAT Gateways, IAM roles, security groups, Regional WAF v2, and API Gateway | Week 3 |
| **Phase 3: Database & Compute Provisioning** | Provision RDS MySQL Multi-AZ, ElastiCache Redis, DynamoDB tables, S3 private buckets with VPC Endpoints, EC2 Launch Template, and ASG | Week 4 |
| **Phase 4: Frontend Amplify & Backend Bootstrap** | Configure `amplify.tf`, write `user_data.tftpl` bootstrap script for EC2 (Node.js 20, PM2, S3 artifact pull), verify full-stack HTTP/HTTPS connectivity | Week 5 |
| **Phase 5: CI/CD Pipeline & Verification** | Build GitHub Actions workflows for Amplify frontend release and ASG rolling instance refresh; run smoke tests and write documentation | Week 6 |

---

### 5. Budget Estimation

Estimated monthly cost breakdown using the AWS Pricing Calculator (us-east-1 region), projected for **40,000 registered users** (~8,000 DAU, ~7.2M API requests/month, ~2 TB frontend bandwidth/month):

#### Tier 1 — Compute & Load Balancing

| AWS Service | Configuration Details | Monthly Cost (USD) |
| :--- | :--- | :--- |
| **EC2 Auto Scaling** | 2–4 × `t3.medium` instances (avg 2.5 running) | ~$75.92 |
| **EBS Storage (gp3)** | 30 GB per instance × 2.5 instances | ~$6.00 |
| **Application Load Balancer** | 1 ALB (fixed + ~2.5 LCU data processing) | ~$31.03 |
| **NAT Gateways** | 2 NAT Gateways (1/AZ, always-on) + ~50 GB processed | ~$67.95 |
| **Public IPv4 Addresses** | 4 IPs (2 NAT EIPs + 2 ALB IPs) | ~$14.60 |
| **VPC Gateway Endpoints** | S3 & DynamoDB (free of charge) | $0.00 |
| | **Tier 1 Subtotal** | **~$195.50** |

#### Tier 2 — Database & Storage

| AWS Service | Configuration Details | Monthly Cost (USD) |
| :--- | :--- | :--- |
| **RDS MySQL Multi-AZ** | `db.t4g.micro`, 20 GB gp3 storage, 7-day backup | ~$25.66 |
| **ElastiCache Redis** | `cache.t4g.micro`, 2-node (primary + replica) | ~$23.36 |
| **DynamoDB (5 tables)** | On-Demand, PITR enabled (~12M writes + 48M reads/month) | ~$15.75 |
| **Amazon S3** | 3 private buckets, ~20 GB storage | ~$1.50 |
| | **Tier 2 Subtotal** | **~$66.27** |

#### Tier 3 — Security, Frontend Hosting & Monitoring

| AWS Service | Configuration Details | Monthly Cost (USD) |
| :--- | :--- | :--- |
| **AWS WAF v2 (Regional)** | 1 Web ACL + 3 Managed Rule Groups + 1 Rate Limit rule + 7.2M requests | ~$13.32 |
| **AWS Amplify Hosting** | React/Vite SPA, ~2,000 GB bandwidth @ $0.15/GB | ~$300.00 |
| **API Gateway HTTP API** | 7.2M requests @ $1.00/1M | ~$7.20 |
| **VPC Interface Endpoints (SSM)** | 3 endpoints × 2 AZs = 6 ENIs @ $0.01/hr | ~$43.80 |
| **CloudWatch** | 3 alarms + ~10 GB log ingestion | ~$5.30 |
| **IAM** | Roles & Policies | $0.00 |
| | **Tier 3 Subtotal** | **~$369.62** |

#### Total Estimated Monthly Cost

| | Monthly Cost (USD) |
| :--- | :--- |
| Tier 1 — Compute & Load Balancing | $195.50 |
| Tier 2 — Database & Storage | $66.27 |
| Tier 3 — Security, Frontend & Monitoring | $369.62 |
| **Total Estimated Cost** | **~$631.39 / month** |

*Note: The largest cost driver is Amplify Hosting bandwidth (~$300, 47.5% of total). Switching to CloudFront + S3 static hosting can reduce bandwidth cost to ~$170/month. Compute Savings Plans and Reserved Instances for EC2/RDS/Redis can further reduce costs by 30–50%. During workshop testing, costs can be minimized to near $0 by tearing down resources via `terraform destroy`.*

---

### 6. Risk Assessment & Mitigation

| Identified Risk | Impact | Probability | Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **Mixed Content Browser Security Block** | High | High | Implemented **AWS API Gateway HTTP API** to serve HTTPS endpoint, forwarding traffic securely to ALB. |
| **Malicious DDoS or Bot Attacks** | High | Medium | Provisioned **Regional AWS WAF v2** with rate-limiting rules (2,000 req/5min) and OWASP Top 10 rules. |
| **EC2 Server Crash or High Traffic** | High | Medium | Configured **EC2 Auto Scaling Group** with CloudWatch alarms to auto-scale instances and health-check targets. |
| **Unexpected AWS Cloud Costs** | Medium | Low | Set up CloudWatch billing alerts and automated teardown script via `terraform destroy`. |

---

### 7. Expected Outcomes

1. **100% Infrastructure as Code**: Entire enterprise cloud environment provisioned in ~15 minutes using `terraform apply`.
2. **Zero-Downtime CI/CD Pipelines**: Automated deployment workflows for both Amplify frontend and backend ASG rolling refresh.
3. **Production Security Standards**: Private EC2 instances, SSM Session Manager shell access, Regional WAF protection, and end-to-end HTTPS.
4. **Complete Workshop Documentation**: Structured Hugo workshop documentation and worklogs for future team expansion and academic reference.