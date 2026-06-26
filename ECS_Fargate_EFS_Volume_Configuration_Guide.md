# Amazon ECS Fargate - EFS Volume Configuration Guide

# Overview

This document explains how Amazon EFS integrates with Amazon ECS Fargate, how to configure an EFS volume in a Task Definition, and how it works internally.

---

# Architecture

```text
                Amazon EFS
          File System (fs-xxxxxxxx)
                    │
            EFS Access Point
                    │
            ECS Task Definition
              Volume Configuration
                    │
             Container Mount Point
                 (/data)
                    │
          ECS Task (Fargate)
                    │
             Application Reads/Writes
```

---

# Why Use EFS?

Amazon ECS Fargate containers use ephemeral storage. If a task stops, any data written inside the container is lost.

Use Amazon EFS when you need:

- Persistent storage
- Shared storage between multiple tasks
- User uploads
- Shared reports
- WordPress media
- Large ML models
- Shared configuration files

Do **not** use EFS for stateless applications such as simple REST APIs or static NGINX websites.

---

# Step 1 - Create Amazon EFS

1. Open **Amazon EFS**
2. Click **Create File System**
3. Enter:
   - Name: `nginx-efs`
4. Create mount targets in every Availability Zone where ECS tasks will run.

Example:

```
Private Subnet A
Private Subnet B
```

---

# Step 2 - Configure Security Group

Create an EFS Security Group.

Inbound Rule

| Protocol | Port | Source |
|----------|------|--------|
| TCP | 2049 | ECS Task Security Group |

Outbound: Allow All

> Never allow TCP 2049 from the Internet.

---

# Step 3 - Create an EFS Access Point (Recommended)

Example

- Access Point Name: `nginx-access-point`
- Root Directory: `/nginx-data`

Benefits

- Better security
- Isolated application directories
- Easier permissions management

---

# Step 4 - Create Volume Configuration in ECS Task Definition

Open:

```
Amazon ECS
→ Task Definitions
→ Create / New Revision
```

Scroll to **Volumes**.

Click **Add Volume**.

Example:

| Field | Value |
|------|-------|
| Volume Name | efs-volume |
| Volume Type | Amazon EFS |
| File System ID | fs-xxxxxxxx |
| Transit Encryption | Enabled |
| Authorization | Enabled |
| Access Point | fsap-xxxxxxxx |

This tells ECS to connect the task to the specified EFS file system.

---

# Step 5 - Configure Container Mount Point

Open your container configuration.

Scroll to **Mount Points**.

Click **Add Mount Point**.

Example

| Field | Value |
|------|-------|
| Source Volume | efs-volume |
| Container Path | /data |
| Read Only | No |

The container can now access EFS using:

```
/data
```

---

# Internal Workflow

```text
Amazon EFS
      │
      ▼
Volume Configuration
      │
      ▼
Container Mount Point (/data)
      │
      ▼
Application
```

When the ECS task starts:

1. ECS connects to EFS.
2. EFS is mounted at `/data`.
3. The container starts.
4. The application reads and writes files in `/data`.

---

# Example

Application writes:

```
/data/report.pdf
```

The file is stored in Amazon EFS.

If the task is replaced, the new task mounts the same EFS and can still access:

```
/data/report.pdf
```

---

# Multiple ECS Tasks

```text
             Amazon EFS
                 │
     ┌───────────┼───────────┐
     ▼           ▼           ▼
 Task 1       Task 2       Task 3
   │             │             │
 /data         /data         /data
```

All tasks share the same files.

---

# NGINX Example

Without EFS

```
Docker Image
 └── index.html
```

Files are packaged inside the image.

No persistent storage is required.

With EFS

```
Docker Image
      │
      ▼
NGINX
      │
      ▼
/data
      │
      ▼
Amazon EFS
```

Store uploads, reports, or shared files in `/data`.

---

# Best Practices

- Use an Access Point instead of the EFS root.
- Enable Transit Encryption.
- Restrict NFS (TCP 2049) to the ECS Task Security Group.
- Use EFS only for persistent shared storage.
- Use Amazon S3 for object storage when a shared file system is unnecessary.

---

# Summary

- ECS volumes define storage available to containers.
- Amazon EFS provides persistent, shared storage.
- Volume Configuration connects ECS to EFS.
- Mount Points expose the EFS inside the container (for example `/data`).
- Stateless applications usually do not require EFS.
