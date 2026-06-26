# Amazon ECR (Elastic Container Registry) - Complete Guide

# Overview

Amazon ECR is a fully managed Docker container registry where container images are stored before they are deployed to Amazon ECS, EKS, or other container platforms.

**Deployment Flow**

```text
Developer
    │
    ▼
Build Docker Image
    │
    ▼
Tag Docker Image
    │
    ▼
Push Image to Amazon ECR
    │
    ▼
Amazon ECS Task Definition
    │
    ▼
Amazon ECS Service
```

---

# Prerequisites

- AWS Account
- IAM permissions for ECR
- Docker installed
- AWS CLI v2 configured

Verify:

```bash
aws sts get-caller-identity
aws --version
docker --version
```

---

# Step 1 – Create a Private ECR Repository

AWS Console

```
Amazon ECR
→ Private Registry
→ Repositories
→ Create Repository
```

Configuration

| Setting | Example |
|---------|---------|
| Visibility | Private |
| Repository Name | nginx-demo |
| Tag Mutability | Immutable (Recommended) |
| Scan on Push | Enabled |
| Encryption | AES-256 (Default) or KMS |

Click **Create Repository**.

Example repository URI:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com/nginx-demo
```

---

# Step 2 – Create a Sample Application

Create `index.html`

```html
<h1>Hello from Amazon ECS Fargate!</h1>
```

Create `Dockerfile`

```dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

---

# Step 3 – Build the Docker Image

```bash
docker build -t nginx-demo:v1 .
```

Verify:

```bash
docker images
```

---

# Step 4 – Authenticate Docker to Amazon ECR

Replace REGION with your AWS Region.

```bash
aws ecr get-login-password --region ap-south-1 \
| docker login \
--username AWS \
--password-stdin 123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

Expected output:

```text
Login Succeeded
```

---

# Step 5 – Tag the Image

```bash
docker tag nginx-demo:v1 \
123456789012.dkr.ecr.ap-south-1.amazonaws.com/nginx-demo:v1
```

Verify:

```bash
docker images
```

---

# Step 6 – Push the Image

```bash
docker push 123456789012.dkr.ecr.ap-south-1.amazonaws.com/nginx-demo:v1
```

Verify in the AWS Console that the image and tag are visible.

---

# Step 7 – Copy the Image URI

Use this URI in the ECS Task Definition.

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com/nginx-demo:v1
```

---

# Updating the Application

1. Modify the application.
2. Build a new image.

```bash
docker build -t nginx-demo:v2 .
```

3. Tag the image.

```bash
docker tag nginx-demo:v2 \
123456789012.dkr.ecr.ap-south-1.amazonaws.com/nginx-demo:v2
```

4. Push the image.

```bash
docker push 123456789012.dkr.ecr.ap-south-1.amazonaws.com/nginx-demo:v2
```

5. Create a new ECS Task Definition revision and update the ECS Service.

---

# Best Practices

- Use private repositories for production.
- Enable image scan on push.
- Use immutable tags.
- Use meaningful version tags (v1.0.0, v1.0.1).
- Do not use `latest` for production deployments.
- Configure lifecycle policies to delete old images.
- Restrict IAM permissions using least privilege.

---

# Troubleshooting

| Issue | Resolution |
|--------|------------|
| no basic auth credentials | Run ECR login again |
| AccessDeniedException | Check IAM permissions |
| Repository not found | Verify repository name and region |
| Image push denied | Check repository URI and login |

---

# Summary

Amazon ECR stores your Docker images securely. The typical workflow is:

```text
Create Repository
      ↓
Build Docker Image
      ↓
Authenticate Docker
      ↓
Tag Image
      ↓
Push Image
      ↓
Use Image URI in ECS Task Definition
      ↓
Deploy ECS Service
```
