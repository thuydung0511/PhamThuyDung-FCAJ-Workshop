---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# CloudNote — Serverless App Implementation Workshop

![Workshop screenshots](/images/5-Workshop/image1.png)

Hands-on implementation of my capstone project **CloudNote** — a **serverless notes application** built on AWS Free Tier: a static frontend hosted on **S3**, a CRUD API with **Lambda + API Gateway**, data stored in **DynamoDB**, monitored by **CloudWatch** and audited by **CloudTrail**. A bonus section adds a horizontally-scalable web tier with **VPC, EC2, ALB and Auto Scaling**. The CloudNote repository is being prepared — link will be updated soon.

**Region:** `ap-southeast-1`  
**Scope:** Serverless core (Part A): IAM, DynamoDB `Notes`, Lambda `notes-api`, API Gateway HTTP API, S3 static hosting, CloudWatch dashboard/alarm, CloudTrail. Bonus web tier (Part B): VPC, Launch Template, Target Group, ALB, Auto Scaling.

#### Contents

1. [Workshop overview](5.1-Workshop-overview/)
2. [EC2 web tier & Auto Scaling](5.2-EC2-Fleet/)
3. [S3 static app hosting](5.3-S3-Hosting/)
4. [IAM roles & policies](5.4-IAM/)
5. [GitHub OIDC → AWS (CI authentication)](5.5-GitHub-OIDC/)
6. [DynamoDB, Lambda & API Gateway backend](5.6-CodeDeploy/)
7. [Monitoring & auditing (CloudWatch, CloudTrail)](5.7-Async-Processing/)
8. [VPC — serverless private networking](5.8-VPC-Networking/)
9. [App demo](5.9-App-Demo/)
10. [Resource cleanup (documented)](5.10-Cleanup/)