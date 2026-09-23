---
title: "EC2 Web Tier & Auto Scaling"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Goal

Provision the **Part B web tier**: an **EC2 Launch Template** plus an **Auto Scaling Group (ASG)** behind an Application Load Balancer, so the CloudNote web tier can scale horizontally. This is a classic, scalable web-serving pattern (VPC, EC2, ELB, Auto Scaling) complementing the serverless core of Part A.

## Step 1 — Create the Launch Template

1. **EC2 → Launch Templates → Create launch template** in `ap-southeast-1`.
2. **Name:** `cloudnote-web-template`.
3. **AMI:** **Amazon Linux 2023** (Free tier eligible).
4. **Instance type:** `t2.micro` (or `t3.micro` if available in your region).
5. **Key pair:** create `cloudnote-key`, download and store the `.pem`.

![Create launch template](/images/5-Workshop/image4.png)

6. **Network settings → Security groups → Create security group**:
   - Name: `cloudnote-web-sg`
   - Inbound rules: `HTTP (80)` from `0.0.0.0/0`, `SSH (22)` from your IP only.
7. **Advanced details → User data** — install a web server and show the Instance ID:

```bash
#!/bin/bash
dnf install -y httpd
systemctl enable httpd
systemctl start httpd
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
echo "<h1>CloudNote Web Tier</h1><p>Phục vụ bởi instance: $INSTANCE_ID</p>" > /var/www/html/index.html
```

8. **Create launch template**.

## Step 2 — Target Group & Application Load Balancer

1. **EC2 → Target Groups → Create target group**.
2. **Target type:** `Instances` · **Name:** `cloudnote-tg` · **Protocol:** `HTTP:80` · **VPC:** `cloudnote-vpc`.
3. **Health check path:** `/` → create group (do not register instances manually — the ASG does it).
4. **EC2 → Load Balancers → Create load balancer → Application Load Balancer**:
   - **Name:** `cloudnote-alb` · **Scheme:** Internet-facing.
   - **VPC:** `cloudnote-vpc`, select **both public subnets** (ALB requires ≥ 2 AZ).
   - **Security group:** `cloudnote-web-sg` (or an ALB SG allowing HTTP 80 from `0.0.0.0/0`).
   - **Listener:** `HTTP:80` → forward to `cloudnote-tg`.
5. Wait until state is **Active**, copy the ALB **DNS name**.

![Create ALB](/images/5-Workshop/image5.png)

## Step 3 — Create the Auto Scaling Group

1. **EC2 → Auto Scaling Groups → Create Auto Scaling group**.
2. **Name:** `cloudnote-asg` · **Launch template:** `cloudnote-web-template`.
3. **VPC:** `cloudnote-vpc`, select **both public subnets**.
4. **Load balancing:** attach to existing load balancer → target group `cloudnote-tg`.
5. **Health checks:** enable **ELB health checks** (not only EC2).
6. **Group size:** Desired = `2`, Minimum = `1`, Maximum = `2`.
7. (Optional) **Target tracking** scaling policy on average CPU utilization at 50%.
8. **Create Auto Scaling group**.

![Create ASG](/images/5-Workshop/image5.png)

## Step 4 — Verify the fleet

1. Wait 2–3 minutes for both instances to reach **Healthy** in the target group.
2. Open the ALB DNS name; refresh several times — the Instance ID shown must alternate (load balancing works).
3. This is the **second demo product** of the capstone, easy to record for the report.

## Expected outcome

- Repeatable web-tier instances from a single launch template
- ASG keeps 1–2 instances healthy and scales with load
- ALB distributes HTTP traffic across both AZs

## Troubleshooting

| Issue | Check |
|---|---|
| Targets show **Unhealthy** | Security group allows HTTP 80 from `0.0.0.0/0`; User Data script ran (`systemctl status httpd`) |
| ASG never reaches desired capacity | vCPU limit/quotas, instance type availability, wrong subnets |
| ALB DNS does not alternate | Browser cache; check both instances are in service |