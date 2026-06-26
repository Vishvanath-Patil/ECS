# AWS ECS Fargate Deployment Workflows

## Overview

This document explains the two common deployment workflows used with Amazon ECS Fargate:

1. **First-Time Deployment** – Create the complete infrastructure and deploy the application.
2. **Existing Setup Deployment** – Reuse the existing infrastructure and deploy a new application version.

---

# Workflow 1 – First-Time Deployment

```text
Developer
    |
    v
Develop Application
    |
    v
Create Dockerfile
    |
    v
Build Docker Image
    |
    v
Test Container Locally
    |
    v
Create VPC
    |
    v
Create Public & Private Subnets
    |
    v
Create Internet Gateway
    |
    v
Create NAT Gateway
    |
    v
Configure Route Tables
    |
    v
Create Security Groups
    |
    v
Create IAM Roles
    |
    v
Create Private Amazon ECR Repository
    |
    v
Push Docker Image to Amazon ECR
    |
    v
Create ECS Cluster
    |
    v
(Optional) Create Amazon EFS
    |
    v
Create Task Definition
    |
    v
Create Application Load Balancer
    |
    v
Create Target Group
    |
    v
Create ECS Service
    |
    v
Launch Tasks
    |
    v
Health Check Passed
    |
    v
Application Available
```

## Resources Created

| Resource | Created |
|----------|---------|
| VPC | Yes |
| Public & Private Subnets | Yes |
| Internet Gateway | Yes |
| NAT Gateway | Yes |
| Route Tables | Yes |
| Security Groups | Yes |
| IAM Roles | Yes |
| Amazon ECR Repository | Yes |
| ECS Cluster | Yes |
| Task Definition | Yes |
| Application Load Balancer | Yes |
| Target Group | Yes |
| ECS Service | Yes |
| Amazon EFS | Optional |

### Why EFS is Optional

Use EFS only if the application needs persistent or shared storage.

Examples:
- User uploads
- Shared files
- WordPress media
- Shared reports

Not required for:
- NGINX static website
- REST APIs
- Stateless microservices

---

# Workflow 2 – Existing Setup Deployment

```text
Developer
    |
    v
Modify Application
    |
    v
Build Docker Image
    |
    v
Tag Docker Image
    |
    v
Push Image to Existing Amazon ECR
    |
    v
Create New Task Definition Revision
    |
    v
Update Existing ECS Service
    |
    v
Launch New Tasks
    |
    v
ALB Health Checks
    |
    v
Traffic Shifted to New Tasks
    |
    v
Old Tasks Stopped
    |
    v
Deployment Completed
```

## Resources Reused

- VPC
- Subnets
- Internet Gateway
- NAT Gateway
- Route Tables
- Security Groups
- IAM Roles
- ECS Cluster
- ALB
- Target Group
- EFS (if configured)

## Resources Updated

- Docker Image
- Amazon ECR Image Tag
- Task Definition Revision
- ECS Service
- ECS Tasks
- CloudWatch Log Streams

---

# Production CI/CD Flow

```text
Developer
   |
Git Push
   |
GitHub / GitLab
   |
CI/CD Pipeline
(Jenkins / GitHub Actions)
   |
Build Docker Image
   |
Security Scan
   |
Push Image to Amazon ECR
   |
Register New Task Definition Revision
   |
Update ECS Service
   |
Launch New Tasks
   |
Health Checks
   |
Traffic Shift
   |
Old Tasks Removed
```

---

# Comparison

| Activity | First-Time | Existing |
|----------|------------|----------|
| Create VPC | Yes | No |
| Create Subnets | Yes | No |
| Create IAM Roles | Yes | No |
| Create ECR | Yes | No |
| Push Docker Image | Yes | Yes |
| Create ECS Cluster | Yes | No |
| Create Task Definition | Yes | New Revision |
| Create ALB | Yes | No |
| Create ECS Service | Yes | No |
| Update ECS Service | No | Yes |
| Rolling Deployment | No | Yes |

---

# Best Practices

- Build infrastructure once.
- Reuse infrastructure for every deployment.
- Create a new Task Definition revision for each release.
- Use immutable image tags.
- Keep ECS tasks in private subnets.
- Store secrets in AWS Secrets Manager.
- Enable CloudWatch Logs.
- Use EFS only when persistent storage is required.
