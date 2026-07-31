---
title: "Worklog Week 4"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives

- Self-study Terraform modules and Docker containerization concepts
- Learn GitHub Actions CI/CD automation and implement automated deployment pipelines for the team project
- Learn AWS architecture diagramming standards and design the 3-tier infrastructure layout

### Tasks to Complete This Week

| Day | Task | Start Date | End Date | Resources |
| --- | ---- | ---------- | -------- | --------- |
| Mon | - Self-study advanced Terraform concepts: reusable modules, state management, and environment variables <br> - Deepen Docker knowledge: multi-stage builds and container configuration patterns | July 6, 2026 | July 6, 2026 | [Terraform Beginner to Pro](https://courses.devopsdirective.com/terraform-beginner-to-pro/lessons), [Docker Beginner to Pro](https://courses.devopsdirective.com/docker-beginner-to-pro/lessons) |
| Tue | - Practice Docker containerization locally: experiment with writing `Dockerfile` and local container execution <br> - Evaluate containerizing the LMS application (determined Docker will not be used in the final project production setup to keep deployment streamlined) | July 7, 2026 | July 7, 2026 | [Docker Beginner to Pro](https://courses.devopsdirective.com/docker-beginner-to-pro/lessons) |
| Wed | - Self-study GitHub Actions CI/CD: workflows, jobs, steps, runners, triggers (`push`, `pull_request`), and repository secrets <br> - Write initial `.github/workflows/ci.yml` script to test code linting and automated build checks | July 8, 2026 | July 8, 2026 | [GitHub Actions Beginner to Pro](https://courses.devopsdirective.com/github-actions-beginner-to-pro/lessons) |
| Thu | - Implement GitHub Actions CI/CD pipelines into team repository: <br> &nbsp;&nbsp;• Frontend pipeline: automated build and release workflow <br> &nbsp;&nbsp;• Backend pipeline: automated testing, build, and deployment trigger workflow <br> - Configure GitHub Repository Secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, etc.) | July 9, 2026 | July 9, 2026 | [GitHub Actions Beginner to Pro](https://courses.devopsdirective.com/github-actions-beginner-to-pro/lessons) |
| Fri | - Learn AWS Architecture Diagramming guidelines: official AWS Architecture Icons, subnet tiering, and flow arrows <br> - Draft preliminary 3-tier AWS architecture diagram for the team's LMS project (VPC, public/private subnets, ALB, ASG, RDS, Redis, DynamoDB, S3) | July 10, 2026 | July 10, 2026 | [AWS Architecture Icons](https://aws.amazon.com/architecture/icons/) |
| Sat | - Integrate Terraform automation checks into GitHub Actions (`terraform fmt`, `terraform validate`, `terraform plan` on PRs) <br> - Review architecture diagram with team members and adjust resource placements and security boundaries | July 11, 2026 | July 11, 2026 | |
| Sun | - Document GitHub Actions CI/CD workflow and setup instructions in team project repository <br> - Verify successful execution of GitHub Actions workflows on repository push <br> - Summarize week 4 deliverables and plan objectives for Week 5 | July 12, 2026 | July 12, 2026 | |

### Week 4 Results

- Deepened understanding of Terraform modular architecture and Docker containerization fundamentals
- Evaluated Docker containerization locally (decided to keep standard runtime deployment on EC2 for the final project)
- Successfully designed and implemented automated GitHub Actions CI/CD pipelines for frontend and backend in the team repository
- Configured repository secrets and automated Terraform validation workflows on GitHub Actions
- Mastered AWS architecture diagramming standards and completed the official 3-tier AWS infrastructure diagram for the project
