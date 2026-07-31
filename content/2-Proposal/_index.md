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

### 5. Budget Estimation & Cost Framework

During the initial proposal phase, precise cloud costs depend on actual user adoption, peak traffic patterns, and chosen resource sizes. Below is a flexible cost estimation framework categorized by architectural tiers and operating scales.

#### Key Cost Drivers

1. **Compute & Load Balancing (Tier 1)**: EC2 Auto Scaling instances (`t3.medium`/`t4g.small`), Application Load Balancer (ALB), and NAT Gateways.
2. **Database & Storage (Tier 2)**: Multi-AZ RDS (MySQL), ElastiCache (Redis), DynamoDB (On-Demand), and S3 object storage.
3. **Security, Delivery & Edge (Tier 3)**: AWS Amplify / CloudFront CDN bandwidth, Regional AWS WAF v2, API Gateway HTTP API, and SSM Interface Endpoints.

#### Estimated Monthly Budget by Operating Scale

| Environment / Scale | Estimated Monthly Range | Key Assumptions & Resource Allocation |
| :--- | :--- | :--- |
| **Development / Lab Testing** | **~$0 – $30 / month** | Ephemeral resources provisioned on demand via `terraform apply` and destroyed via `terraform destroy` when inactive. Fits mostly within AWS Free Tier. |
| **Staging / Small Production** | **~$150 – $350 / month** | Single-AZ / minimal Multi-AZ setup (`t4g.micro` instances), reduced NAT Gateway footprint, and moderate CDN traffic (<500 GB/month). |
| **Full Production Scale** | **~$500 – $800 / month** | Multi-AZ high availability (~8,000 daily active user, 40,000 registered users), auto-scaling compute (2–4 instances), full WAF protection, and ~2 TB CDN bandwidth. |

#### Cost Control & Optimization Strategies

- **Bandwidth Optimization**: Utilizing AWS CloudFront + S3 static hosting as an alternative to Amplify Hosting can reduce CDN data transfer costs by up to 45%.
- **Savings Plans & Reserved Instances**: Committing to 1-year or 3-year Compute Savings Plans for EC2, RDS, and ElastiCache provides 30%–50% cost savings for steady-state production workloads.
- **Automated Resource Teardown**: In non-production environments, Terraform automation enables `terraform destroy` after testing sessions to eliminate idle instance charges.

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