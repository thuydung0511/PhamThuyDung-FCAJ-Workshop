---
title: "VPC & web tier networking"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Goal

Create a **custom VPC** for the Part B web tier (`cloudnote-vpc`) with 2 public subnets across 2 Availability Zones and an Internet Gateway — the network foundation for the ALB + ASG EC2 fleet. (The serverless core in Part A needs no VPC.)

## Step 1 — Create the VPC with the wizard

1. **VPC → Create VPC → VPC and more** (wizard creates everything).
2. **Name tag:** `cloudnote-vpc`.
3. **IPv4 CIDR:** `10.0.0.0/16`.
4. **Number of Availability Zones:** `2`.
5. **Number of public subnets:** `2` · **private subnets:** `0` (avoid NAT costs).
6. **NAT gateways:** **None** · **VPC endpoints:** **None**.
7. **Create VPC**.

![Create VPC](/images/5-Workshop/5.8-VPC-Networking/vpc.png)

## Step 2 — Verify the network components

1. **Your VPCs:** `cloudnote-vpc` exists.
2. **Subnets:** 2 public subnets in different AZs.
3. **Internet Gateways:** 1 IGW attached to the VPC.

![Subnets](/images/5-Workshop/5.8-VPC-Networking/subnets.png)

## Step 3 — Use it for the web tier

- Place the Load Balancer + ASG instances in both public subnets ([5.2](5.2-EC2-Fleet/)); the ALB requires ≥ 2 AZs.
- Security group `cloudnote-web-sg` allows HTTP `80` from `0.0.0.0/0` and SSH `22` from your IP.

![Security groups](/images/5-Workshop/5.8-VPC-Networking/security-groups.png)

## Expected outcome

- Dedicated, isolated network for the bonus web tier
- Two AZs for high availability
- Internet reachability via the attached IGW

## Troubleshooting

| Issue | Check |
|-------|-------|
| ALB targets unhealthy | Instances and ALB must share the VPC; SG allows HTTP 80 |
| No internet on instances | IGW attached to VPC; subnet route table has a default route via IGW |
| ASG can't launch | Subnets selected belong to `cloudnote-vpc` |