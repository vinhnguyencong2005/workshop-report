---
title: "Worklog Week 4"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives

- Deepen knowledge of Terraform and Docker containerization for full-stack application setup
- Self-study GitHub Actions CI/CD automation and implement automated workflows for the team project
- Learn AWS architecture diagramming standards and design the 3-tier infrastructure layout

### Tasks to Complete This Week

| Day | Task | Start Date | End Date | Resources |
| --- | ---- | ---------- | -------- | --------- |
| Mon | - Self-study advanced Terraform concepts: reusable modules, state management, and environment variables <br> - Deepen Docker knowledge: multi-stage builds and `docker-compose.yml` for multi-container orchestration | 06/07/2026 | 06/07/2026 | [Terraform Beginner to Pro](https://courses.devopsdirective.com/terraform-beginner-to-pro/lessons), [Docker Beginner to Pro](https://courses.devopsdirective.com/docker-beginner-to-pro/lessons) |
| Tue | - Dockerize the team's LMS project: write `Dockerfile` for frontend and backend services <br> - Configure `docker-compose.yml` to spin up local Node.js backend, Redis cache, and MySQL database <br> - Test local containerized environment to ensure consistency across dev machines | 07/07/2026 | 07/07/2026 | [Docker Beginner to Pro](https://courses.devopsdirective.com/docker-beginner-to-pro/lessons) |
| Wed | - Self-study GitHub Actions CI/CD: workflows, jobs, steps, runners, triggers (`push`, `pull_request`), and repository secrets <br> - Write initial `.github/workflows/ci.yml` script to test code linting and automated build checks | 08/07/2026 | 08/07/2026 | [GitHub Actions Beginner to Pro](https://courses.devopsdirective.com/github-actions-beginner-to-pro/lessons) |
| Thu | - Implement GitHub Actions CI/CD pipelines into team repository: <br> &nbsp;&nbsp;• Frontend pipeline: automated build and deployment validation <br> &nbsp;&nbsp;• Backend pipeline: automated syntax checks, testing, and deployment preparation <br> - Configure GitHub Repository Secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, etc.) | 09/07/2026 | 09/07/2026 | [GitHub Actions Beginner to Pro](https://courses.devopsdirective.com/github-actions-beginner-to-pro/lessons) |
| Fri | - Learn AWS Architecture Diagramming guidelines: official AWS Architecture Icons, subnet tiering, and flow arrows <br> - Draft preliminary 3-tier AWS architecture diagram for the team's LMS project (VPC, public/private subnets, ALB, ASG, RDS, Redis, DynamoDB, S3) | 10/07/2026 | 10/07/2026 | [AWS Architecture Icons](https://aws.amazon.com/architecture/icons/) |
| Sat | - Integrate Terraform automation checks into GitHub Actions (`terraform fmt`, `terraform validate`, `terraform plan` on PRs) <br> - Review architecture diagram with team members and adjust resource placements and security boundaries | 11/07/2026 | 11/07/2026 | |
| Sun | - Document Docker environment setup and GitHub Actions CI/CD workflow in project repository <br> - Verify successful execution of GitHub Actions workflows on repository push <br> - Summarize week 4 deliverables and plan objectives for Week 5 | 12/07/2026 | 12/07/2026 | |

### Week 4 Results

- Mastered advanced Terraform modular design and Docker multi-stage containerization techniques
- Successfully containerized team LMS application locally using Docker and Docker Compose
- Designed and implemented automated GitHub Actions CI/CD pipelines for frontend and backend in team repository
- Configured repository secrets and automated Terraform validation workflows on GitHub Actions
- Mastered AWS architecture diagramming standards and completed the official 3-tier AWS infrastructure diagram for the project
