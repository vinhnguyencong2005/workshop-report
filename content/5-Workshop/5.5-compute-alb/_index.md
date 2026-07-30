---
title : "Compute & Load Balancing"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

Three components work together to run and expose the application:

```
[ALB] ──→ [ASG: EC2 ×2–4] ──→ [SSM Endpoints]
  │              │
  │         Launch Template
  │         (AMI, user data, IAM, EBS)
  │
  Target Group :3000
  Health check /health
```

| Component | What It Does |
|-----------|-------------|
| **Launch Template** | Defines the EC2 blueprint: AMI, instance type, user data bootstrap, IAM profile, EBS volumes, IMDSv2 |
| **SSM VPC Endpoints** | Enable Session Manager access to EC2 without SSH or bastion hosts |
| **Auto Scaling Group** | Maintains 2–4 instances across two AZs, replaces unhealthy ones, distributes across private app subnets |
| **Application Load Balancer** | Single internet-facing entry point, forwards HTTP :80 → EC2 :3000, health checks at `/health` |

#### Content

1. [Launch Template](5.5.1-launch-template/) — EC2 blueprint with user data and IMDSv2
2. [SSM VPC Endpoints](5.5.2-ssm-endpoints/) — Shell access without SSH
3. [Auto Scaling Group](5.5.3-asg/) — 2–4 instances across two AZs
4. [Application Load Balancer](5.5.4-alb/) — Internet-facing entry point with health checks

---
