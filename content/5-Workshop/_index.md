---
title: "Workshop"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5. </b> "
includeInReport: false
---

# Deploy a Full-Stack LMS on AWS with Terraform

#### Overview

In this workshop, you will deploy a complete **3-tier Learning Management System** on AWS using **Terraform** — Infrastructure as Code. You will provision a VPC with public and private subnets, AWS WAF v2, AWS Amplify Hosting, API Gateway HTTP API, an Auto Scaling Group of EC2 instances running Node.js, RDS MySQL with Multi-AZ, ElastiCache Redis, DynamoDB, S3, and more — all from `.tf` files. No clicking in the console. At the end, a single `terraform destroy` cleans everything up.

#### Content

1. [Workshop Overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequiste/)
3. [Networking & Security](5.3-networking-security/)
4. [Data Layer](5.4-data-layer/)
5. [Compute & Load Balancing](5.5-compute-alb/)
6. [Monitoring](5.6-monitoring/)
7. [Deploy & Verify](5.7-deploy-verify/)
8. [CI/CD](5.8-cicd/)
9. [Clean Up](5.9-cleanup/)