# Accessing Containers in Amazon ECS Fargate

## Overview

Amazon ECS Fargate is a serverless container platform. Unlike EC2-based containers, you **cannot SSH into the underlying host**. AWS provides several secure methods to access your application or troubleshoot containers.

---

# Method 1: Application Load Balancer (Recommended)

Use this to access the application.

```text
User
  │
  ▼
Application Load Balancer
  │
  ▼
ECS Service
  │
  ▼
ECS Task (Container)
```

Example:

```
http://my-nginx-alb.ap-south-1.elb.amazonaws.com
```

**Use case:** Production web applications.

---

# Method 2: Public IP (Development Only)

During ECS Service creation:

```
Networking
→ Assign Public IP
→ ENABLED
```

The task receives a public IP.

Example:

```
http://3.110.xxx.xxx
```

> **Not recommended for production.**

---

# Method 3: ECS Exec (Recommended for Shell Access)

Equivalent to:

```bash
docker exec -it <container> /bin/bash
```

## Prerequisites

- Enable ECS Exec on the ECS Service.
- AWS CLI installed.
- Session Manager Plugin installed.
- Task Execution Role with SSM permissions.

## Find Cluster

```
Amazon ECS
→ Clusters
→ nginx-cluster
```

## Find Task ID

```
Cluster
→ Tasks
→ Running Task
```

## Execute Command

```bash
aws ecs execute-command \
  --cluster nginx-cluster \
  --task TASK_ID \
  --container nginx \
  --interactive \
  --command "/bin/sh"
```

If Bash exists:

```bash
aws ecs execute-command \
  --cluster nginx-cluster \
  --task TASK_ID \
  --container nginx \
  --interactive \
  --command "/bin/bash"
```

---

# Useful Commands Inside the Container

```bash
pwd
ls -l
env
cat /etc/os-release
ps -ef
df -h
free -m
```

---

# Method 4: CloudWatch Logs

View application logs.

```
CloudWatch
→ Log Groups
→ /ecs/nginx
```

Use this for troubleshooting without opening a shell.

---

# Method 5: Bastion Host + ECS Exec

For private VPCs:

```text
Laptop
   │
SSH
   │
Bastion Host
   │
AWS CLI
   │
ECS Exec
   │
Container
```

---

# Method 6: AWS Systems Manager

ECS Exec uses AWS Systems Manager (SSM) to create a secure session. No SSH daemon is required inside the container.

---

# Comparison

| Method | Production | Purpose |
|---------|------------|---------|
| ALB | Yes | Access application |
| Public IP | No | Development/testing |
| ECS Exec | Yes | Shell access |
| CloudWatch Logs | Yes | Log analysis |
| Bastion + ECS Exec | Yes | Private environments |

---

# Best Practices

- Use ALB for application traffic.
- Use ECS Exec for debugging.
- Keep tasks in private subnets.
- Avoid assigning public IPs in production.
- Enable CloudWatch Logs.
- Restrict IAM permissions using least privilege.

---

# Production Architecture

```text
Users
   │
   ▼
Application Load Balancer
   │
   ▼
ECS Service
   │
   ▼
ECS Task
   ├── ECS Exec
   ├── CloudWatch Logs
   └── Amazon EFS (Optional)
```

## Summary

- Access the application through the ALB.
- Access the container shell using ECS Exec.
- Use CloudWatch Logs for monitoring.
- Use Public IP only for temporary development.
