
# AWS ECS Fargate End-to-End Deployment Guide (NGINX Example)

> Beginner-friendly, GitHub-ready guide to deploy an NGINX container on Amazon ECS Fargate.

---

# 1. Architecture

```text
Developer
    |
Docker Build
    |
Private Amazon ECR
    |
ECS Cluster (Fargate)
    |
Task Definition
    |
ECS Service
    |
Application Load Balancer
    |
Internet Users
```

---

# 2. Prerequisites

- AWS Account
- AWS CLI v2 configured
- Docker installed
- IAM permissions
- Existing VPC or create one
- Two Public Subnets
- Two Private Subnets

Verify

```bash
aws sts get-caller-identity
aws configure list
docker --version
```

---

# 3. Create VPC

Recommended CIDR

```
10.0.0.0/16
```

Create

- Public Subnet A
- Public Subnet B
- Private Subnet A
- Private Subnet B

Attach

- Internet Gateway
- NAT Gateway
- Route Tables

**Note:** Run ECS tasks in private subnets.

---

# 4. Create Security Groups

## ALB SG

Inbound
- TCP 80 from Internet

Outbound
- All

## ECS Task SG

Inbound
- TCP 80 from ALB SG

Outbound
- All

---

# 5. Create IAM Roles

## Task Execution Role

Attach policy:

- AmazonECSTaskExecutionRolePolicy

Purpose

- Pull images from ECR
- Send logs to CloudWatch

## Task Role

Used by your application to access AWS services like S3 or Secrets Manager.

---

# 6. Create Private ECR

Repository Name

```
nginx-demo
```

Enable image scan on push.

Login

```bash
aws ecr get-login-password --region REGION | docker login --username AWS --password-stdin ACCOUNT.dkr.ecr.REGION.amazonaws.com
```

---

# 7. Create Docker Image

Dockerfile

```dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

index.html

```html
<h1>Hello from ECS Fargate!</h1>
```

Build

```bash
docker build -t nginx-demo:v1 .
```

Tag

```bash
docker tag nginx-demo:v1 ACCOUNT.dkr.ecr.REGION.amazonaws.com/nginx-demo:v1
```

Push

```bash
docker push ACCOUNT.dkr.ecr.REGION.amazonaws.com/nginx-demo:v1
```

---

# 8. Create ECS Cluster

Console → ECS → Clusters → Create

- Cluster Name: nginx-cluster
- Infrastructure: AWS Fargate
- Namespace: Optional

---

# 9. Create Task Definition

Family: nginx-task

Launch Type: Fargate

OS: Linux

Network Mode: awsvpc

CPU: 0.5 vCPU

Memory: 1 GB

Execution Role: ecsTaskExecutionRole

Task Role: ecs-nginx-task-role

## Container

Name

```
nginx
```

Image URI

```
ACCOUNT.dkr.ecr.REGION.amazonaws.com/nginx-demo:v1
```

Port

```
80
```

Environment Variable

```
APP_ENV=prod
```

Logs

Driver: awslogs

Log Group

```
/ecs/nginx
```

Health Check

```bash
CMD-SHELL curl -f http://localhost/ || exit 1
```

---

# 10. Optional EFS

Create EFS.

Create mount targets.

Create access point.

Add volume in Task Definition.

Mount path:

```
/data
```

---

# 11. Create Application Load Balancer

Scheme

- Internet-facing

Listener

- HTTP 80

Target Group

- IP target type
- Port 80
- Health Check /

---

# 12. Create ECS Service

Application Type

- Service

Task Definition

- nginx-task

Service Name

```
nginx-service
```

Service Type

- Replica

Desired Tasks

```
2
```

Networking

- Private Subnets
- ECS Task SG
- Public IP Disabled

Attach ALB.

Create Service.

---

# 13. Verify

Check:

- Tasks Running
- Target Group Healthy
- Open ALB DNS
- See "Hello from ECS Fargate!"

---

# 14. Update Application

Edit index.html

Rebuild

```bash
docker build -t nginx-demo:v2 .
docker tag nginx-demo:v2 ACCOUNT.dkr.ecr.REGION.amazonaws.com/nginx-demo:v2
docker push ACCOUNT.dkr.ecr.REGION.amazonaws.com/nginx-demo:v2
```

Register new Task Definition revision.

Update ECS Service.

Observe rolling deployment.

---

# 15. Troubleshooting

| Issue | Check |
|-------|-------|
| CannotPullContainerError | ECR permissions |
| Task stopped | CloudWatch Logs |
| ALB unhealthy | Health check path |
| EFS mount failed | SG port 2049 |
| No Internet | NAT Gateway |

---

# 16. Best Practices

- Use private subnets.
- Keep ALB public, ECS private.
- Store secrets in Secrets Manager.
- Enable CloudWatch Logs.
- Enable image scanning.
- Use immutable image tags.
- Use minimum 2 tasks.
- Configure Auto Scaling.
- Apply least-privilege IAM.

---

# Deployment Flow

```text
Create VPC
      ↓
Create IAM Roles
      ↓
Create Private ECR
      ↓
Build Docker Image
      ↓
Push Image
      ↓
Create ECS Cluster
      ↓
Create Task Definition
      ↓
(Optional) Configure EFS
      ↓
Create ALB
      ↓
Create ECS Service
      ↓
Verify Deployment
      ↓
Update Image
      ↓
Rolling Deployment
```
