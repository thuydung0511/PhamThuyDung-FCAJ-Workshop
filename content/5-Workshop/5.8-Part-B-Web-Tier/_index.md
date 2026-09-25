---
title: "Part B: VPC, ALB, Auto Scaling"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Goal

A separate exercise, not part of the CloudNote architecture: build a web tier of 2 EC2 instances in 2 Availability Zones, behind an Application Load Balancer and managed by an Auto Scaling Group, then verify load balancing. All resources were deleted after testing ([5.9](../5.9-cleanup-cost/)).

![Part B architecture](/images/5-Workshop/architecture-partb.png)

*Figure: Part B architecture: VPC 10.0.0.0/16 with 2 public subnets in 2 AZs, ALB in front of 2 EC2 instances managed by an Auto Scaling Group (min 1, desired 2, max 2).*

## Steps

### 0. Preparation

- Checked that the Zero-Spend Budget and its email alerts were active.
- Granted VPC, EC2 and ELB permissions to `cloudnote-dev` through IAM group `cloudnote-network-group` ([5.2](../5.2-iam/)).

### 1. VPC (B1)

1. **VPC → Create VPC → VPC and more**, name `cloudnote-vpc`, CIDR `10.0.0.0/16`.
2. 2 Availability Zones, **2 public subnets, 0 private subnets**, **no NAT Gateway**.
3. Result: subnets `...-public1-ap-southeast-1a` and `...-public2-ap-southeast-1b`, Internet Gateway `cloudnote-igw`, route table `cloudnote-rtb-public`.
4. Enabled **Auto-assign public IPv4** on both subnets: without a NAT Gateway, EC2 needs a public IP to download packages.

### 2. Launch Template (B2)

1. **EC2 → Launch Templates → Create**, name `cloudnote-web-template`.
2. AMI **Amazon Linux 2023**, instance type **t3.micro**, key pair `cloudnote-key`.
3. Security group `cloudnote-web-sg`:
   - HTTP 80 from anywhere.
   - SSH 22 only from my own IP.
4. **User Data** installs Apache and shows the instance's Instance ID; I also changed it to:
   - Get an **IMDSv2** token before reading the Instance ID metadata.
   - Declare **charset UTF-8** so Vietnamese text displays correctly.

### 3. Target Group and ALB (B3)

1. Target Group `cloudnote-tg`: HTTP:80, health check `/`.
2. ALB `cloudnote-alb`: Internet-facing, both public subnets, security group `cloudnote-web-sg`.
3. Listener HTTP:80 forwarding to `cloudnote-tg`.

### 4. Auto Scaling Group (B4)

1. `cloudnote-asg` using Launch Template `cloudnote-web-template`, both public subnets.
2. **Attach to an existing load balancer**, Target Group `cloudnote-tg`.
3. Desired **2** / Min **1** / Max **2**; both EC2 and **ELB health checks** enabled.

### 5. Load-balancing test (B5)

Open `http://<ALB_DNS_NAME>` and reload the page several times, watching the Instance ID shown.

## Issues and fixes

| Issue | Fix |
|-------|-----|
| Exceeded the limit of 10 directly attached managed policies for the IAM user | Granted permissions through IAM group `cloudnote-network-group` |
| The VPC wizard defaulted to 0 public / 2 private subnets | Changed to 2 public / 0 private |
| The ALB also had the `default` security group selected | Deselected it, kept only `cloudnote-web-sg` |
| The ASG was initially set to "No load balancer" | Changed to "Attach to an existing load balancer", chose the correct Target Group, enabled ELB health checks |
| The browser timed out on the ALB DNS name although all targets were Healthy | Checked with `nslookup` and `curl` in Command Prompt → the AWS infrastructure was fine; the cause was browser cache/extensions → it worked in a private window |

## Test results

- 2 Availability Zones, one public subnet in each.
- The ASG has 2/2 **Healthy** instances; the ALB returns **HTTP 200** and shows the correct Instance ID.
- On reload, the Instance ID alternates between the 2 instances → load balancing works.
