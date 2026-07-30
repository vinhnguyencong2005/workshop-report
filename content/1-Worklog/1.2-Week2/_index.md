---
title: "Worklog Week 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives

- Explore containerization with Docker and container orchestration on AWS (ECS)
- Get introduced to Infrastructure as Code (IaC) concepts with Terraform and AWS CloudFormation
- Continue completing AWS hands-on lab exercises from the study group curriculum

### Tasks to Complete This Week

| Day | Task | Start Date | End Date | Resources |
| --- | ---- | ---------- | -------- | --------- |
| Mon | - Learn Docker fundamentals: images, containers, Dockerfile, docker-compose <br> - Install Docker Desktop and run first containerized application locally <br> - Understand container vs virtual machine architecture differences | 22/06/2026 | 22/06/2026 | [Docker Beginner to Pro](https://courses.devopsdirective.com/docker-beginner-to-pro/lessons) |
| Tue | - Lab: **Deploy Application on Docker** — build Docker image, run containers, manage container lifecycle <br> - Lab: **Deploy applications on Amazon ECS** — create ECS cluster, task definitions, and services <br> - Understand the relationship between Docker containers and AWS ECS/Fargate | 23/06/2026 | 23/06/2026 | [000015.awsstudygroup.com](https://000015.awsstudygroup.com/), [000016.awsstudygroup.com](https://000016.awsstudygroup.com/) |
| Wed | - Lab: **Deploying CI/CD with ECS Container** — set up CI/CD pipeline for containerized applications on ECS <br> - Lab: **AWS CloudFormation** — learn IaC concepts, write CloudFormation templates, deploy stacks <br> - Compare CloudFormation (AWS-native) vs Terraform (multi-cloud) approaches | 24/06/2026 | 24/06/2026 | [000017.awsstudygroup.com](https://000017.awsstudygroup.com/), [000037.awsstudygroup.com](https://000037.awsstudygroup.com/) |
| Thu | - Introduction to Terraform: HCL syntax, providers, resources, state files <br> - Install Terraform CLI, write first `main.tf` to provision an S3 bucket <br> - Learn Terraform workflow: `init` → `plan` → `apply` → `destroy` <br> - Lab: **Deploy AWS Backup to the System** — configure automated backup strategies | 25/06/2026 | 25/06/2026 | [Terraform Beginner to Pro](https://courses.devopsdirective.com/terraform-beginner-to-pro/lessons), [000013.awsstudygroup.com](https://000013.awsstudygroup.com/) |
| Fri | - Lab: **Setting up VPC Peering** — connect multiple VPCs and configure routing between them <br> - Lab: **Set up AWS Transit Gateway** — centralized networking hub for multi-VPC architectures <br> - Understand advanced networking patterns: VPC Peering vs Transit Gateway | 26/06/2026 | 26/06/2026 | [000019.awsstudygroup.com](https://000019.awsstudygroup.com/), [000020.awsstudygroup.com](https://000020.awsstudygroup.com/) |
| Sat | - Lab: **Getting Started with AWS Security Hub** — enable security findings aggregation and compliance checks <br> - Lab: **AWS Web Application Firewall** — configure WAF rules to protect web applications <br> - Lab: **Managing Resources with Tags** — organize and manage AWS resources using tagging strategies | 27/06/2026 | 27/06/2026 | [000018.awsstudygroup.com](https://000018.awsstudygroup.com/), [000026.awsstudygroup.com](https://000026.awsstudygroup.com/), [000027.awsstudygroup.com](https://000027.awsstudygroup.com/) |
| Sun | - Practice Terraform: write modules for VPC and EC2, understand variables and outputs <br> - Review & summarize Docker, Terraform, and CloudFormation knowledge <br> - Compare IaC tools and decide on Terraform for the team project <br> - Prepare for the upcoming week | 28/06/2026 | 28/06/2026 | [Terraform Beginner to Pro](https://courses.devopsdirective.com/terraform-beginner-to-pro/lessons) |

### Week 2 Results

- Understood Docker containerization concepts: images, containers, Dockerfile, and container lifecycle management
- Completed container-related labs on AWS: Docker deployment, ECS cluster provisioning, and CI/CD pipeline for containers
- Gained foundational knowledge of Infrastructure as Code (IaC) with both AWS CloudFormation and Terraform
- Wrote first Terraform configurations: S3 bucket, VPC module, and EC2 instance using HCL syntax
- Explored advanced AWS networking: VPC Peering, Transit Gateway, and multi-VPC routing architectures
- Completed security-focused labs: AWS Security Hub, WAF, and resource tagging best practices
- Decided to adopt Terraform as the primary IaC tool for the team project due to its multi-cloud flexibility and modular design
