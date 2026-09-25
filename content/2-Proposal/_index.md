---
title: "Proposal"
date: 2026-08-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# CloudNote — Serverless Notes Application on AWS

## A Full-Stack, Cost-Optimized Serverless Reference Application Built on AWS Free Tier

---

### 1. Executive Summary

**CloudNote** is a full-stack serverless notes application built as the Capstone Project for the *First Cloud AI Journey (FCAJ)* internship at Amazon Web Services Vietnam. The application lets users open a static website (hosted on **S3**), view the list of notes, and create, edit or delete notes entirely through the browser. All data is stored and processed by serverless AWS services — no servers to manage.

The project follows a core principle: *run everything serverless, understand each step, and stay within the AWS Free Tier*. The core delivers a production-shaped path — IAM, DynamoDB, Lambda, API Gateway, S3 static hosting, CloudWatch monitoring and CloudTrail auditing — while a bonus web tier (Part B) demonstrates horizontal scaling with VPC, EC2 Launch Templates, an Application Load Balancer and Auto Scaling.

---

### 2. Problem Statement

#### What's the Problem?

Building and demonstrating a "real" application on the cloud usually requires understanding many moving parts: identity and access, a data store, a compute layer, an API surface, frontend hosting, monitoring and auditing. Beginners often either stay with toy examples or jump straight to containers/Kubernetes without mastering the fundamentals.

#### The Solution

CloudNote is deliberately simple in scope but complete in its lifecycle. One **Lambda** function exposes CRUD operations through an **API Gateway HTTP API**, backed by a **DynamoDB** table. A static frontend on **S3** calls the API. **CloudWatch** and **CloudTrail** provide observability and auditability. A bonus **VPC + EC2 + ALB + Auto Scaling** web tier adds the classic scale-out web serving pattern.

Benefits:

- **100% Serverless Core**: No EC2 to manage for the main application; Lambda and DynamoDB scale automatically.
- **Free Tier Friendly**: Everything in Part A is $0 within Free Tier limits; part B costs ~$0.07–0.15 for a 3-hour ALB window.
- **Fundamentals Covered**: IAM least-privilege, NoSQL data modeling, API design, CORS, monitoring, auditing, and horizontal scaling.
- **Reusable Blueprint**: The same shape applies to any tiny API app (to-do list, surveys, personal blog backend).

---

### 3. Solution Architecture

The main flow is fully serverless; the bonus lab adds a traditional scale-out web tier.

![CloudNote Serverless Architecture](/images/5-Workshop/architecture-cloudnote.png)

#### Main Flow (Part A — Serverless Core)

1. The user opens the static website URL on **S3** and receives `index.html` + `app.js`.
2. `app.js` calls the API via `fetch` to the **API Gateway HTTP API** endpoint (`/notes`).
3. **API Gateway** routes requests to the **Lambda** function `notes-api` (Python 3.12).
4. **Lambda** uses its attached IAM Role (`LambdaNotesExecutionRole`) to read/write the **DynamoDB** table `Notes`.
5. The result is returned through API Gateway and rendered in the browser.
6. **CloudWatch** records logs and raises alarms on errors; **CloudTrail** records every API call in the account for auditing.

#### Bonus Flow (Part B — Web Tier)

- A custom **VPC** with 2 public subnets across 2 Availability Zones.
- An **EC2 Launch Template** (Amazon Linux 2023, t2.micro) with a User Data script that serves an instance-ID page.
- A **Target Group + Application Load Balancer (ALB)** and an **Auto Scaling Group** that keep 1–2 web instances running; refreshing the ALB DNS name alternates the displayed Instance ID.

#### AWS Services Used

- **IAM**: practice user `cloudnote-dev`; execution role `LambdaNotesExecutionRole` + inline policy `NotesTableAccess`.
- **Amazon DynamoDB**: table `Notes` (partition key `noteId`, on-demand capacity).
- **AWS Lambda**: `notes-api` — a single CRUD function (GET/POST/PUT/DELETE).
- **Amazon API Gateway**: HTTP API `notes-http-api` with routes `GET/POST /notes` and `PUT/DELETE /notes/{noteId}`.
- **Amazon S3**: bucket `cloudnote-app-0205568-2026` — static website hosting + public read bucket policy.
- **Amazon CloudWatch**: dashboard `CloudNote-Dashboard`; alarm `notes-api-error-alarm`.
- **AWS CloudTrail**: trail `cloudnote-audit-trail` (management events).
- **VPC / EC2 / ELB / Auto Scaling** (Part B): `cloudnote-vpc`, `cloudnote-web-template`, `cloudnote-tg`, `cloudnote-alb`, `cloudnote-asg`.

---

### 4. Technical Implementation

#### Implementation Steps (Part A)

1. **A0 — IAM user**: create `cloudnote-dev` with console access; attach read/execute-focused policies; never use root.
2. **A1 — DynamoDB**: create table `Notes`, partition key `noteId` (String), **On-demand** capacity.
3. **A2 — IAM Role**: create `LambdaNotesExecutionRole` with `AWSLambdaBasicExecutionRole` + inline `NotesTableAccess` (DynamoDB CRUD scoped to the `Notes` table ARN).
4. **A3 — Lambda**: create `notes-api` (Python 3.12, x86_64), attach the role, implement CRUD, test a POST from the console.
5. **A4 — API Gateway**: build HTTP API `notes-http-api`, add 4 routes to `notes-api`, enable CORS, note the Invoke URL.
6. **A5 — API test**: verify `GET /notes` returns JSON; test POST via browser/CloudShell curl.
7. **A6 — S3 hosting**: create bucket `cloudnote-app-0205568-2026`, enable static website hosting, attach public-read bucket policy, upload `index.html` (frontend calls the Invoke URL).
8. **A7 — E2E test**: open the website endpoint, create and delete a note in the browser.
9. **A8 — CloudWatch**: build dashboard with Lambda `Invocations/Errors/Duration` + DynamoDB `Consumed*Capacity`, create alarm on Lambda Errors.
10. **A9 — CloudTrail**: create trail `cloudnote-audit-trail` (management events, new S3 bucket for logs).

#### Implementation Steps (Part B — optional)

B1 custom VPC (2 AZ, 2 public subnets, no NAT) → B2 launch template `cloudnote-web-template` → B3 target group `cloudnote-tg` + ALB `cloudnote-alb` → B4 ASG `cloudnote-asg` (desired 2, min 1, max 2, ELB health checks) → B5 verify load balancing alternates instance IDs → B6 (advanced) optional Docker + ECS Fargate.

#### Technical Requirements

- AWS Console only (no CLI required); region **ap-southeast-1 (Singapore)**.
- MFA enabled on the root account.
- A new (or low-traffic) account to stay inside Free Tier limits.

---

### 5. Timeline & Milestones

- **Week 1–2**: AWS account setup, MFA, IAM, EC2/VPC labs (foundation for Part B).
- **Week 3**: Explore services, deploy an app on S3 and EC2.
- **Week 4**: Storage (S3, DynamoDB, EBS).
- **Week 5**: CloudWatch monitoring + AWS CLI.
- **Week 6**: Capstone Part A core — DynamoDB, Lambda, IAM role, CRUD backend.
- **Week 7**: API Gateway + CORS, S3 frontend, end-to-end demo, CloudWatch + CloudTrail, (optional) Part B web tier, final report.

---

### 6. Budget Estimation & Cost Optimization

Assumes a **new** AWS account (12-month Free Tier), region `ap-southeast-1`; Part A completed over a few days; Part B run ~3 hours and cleaned up immediately.

| Service | Free Tier limit | Workshop usage | Estimated cost |
|---|---|---|---|
| S3 (Part A) | 5 GB + 20K GET/mo | 1 small bucket | **$0** |
| DynamoDB (on-demand) | 25 GB + 2.5M reads/mo (Always Free) | dozens of items | **$0** |
| AWS Lambda | 1M requests + 400K GB-s/mo | a few hundred calls | **$0** |
| API Gateway (HTTP API) | 1M requests/mo (12 mo) | a few hundred requests | **$0** |
| CloudWatch | 10 custom metrics, 3 dashboards | 1 dashboard + alarm | **$0** |
| CloudTrail | First trail management events free | 1 trail | **$0** |
| EC2 t2/t3.micro (Part B) | 750 hrs/mo | 2 × 3 hrs = 6 hrs | **$0** |
| ALB (Part B) | **No Free Tier** | ~3 hrs | **≈ $0.07–0.15** |
| Data transfer out | 100 GB/mo | few MB | **$0** |
| **Total** | | | **≈ $0–0.20** |

Budget controls: create a **Zero spend budget** in AWS Billing before starting Part B, clean up Part B in the same session, and watch **Cost Explorer** during the workshop week.

---

### 7. Risk Assessment

| Risk Item | Impact | Probability | Mitigation Strategy |
| --- | --- | --- | --- |
| Unexpected charges (ALB) | Medium | Medium | Zero-spend budget; delete Part B in the same session; follow the cleanup order in the workshop |
| CORS errors in the browser | Medium | High | Configure CORS on API Gateway; redeploy `$default` stage; verify with the frontend |
| Lambda 500 errors | Medium | Medium | Check CloudWatch logs (`/aws/lambda/notes-api`) and verify `NotesTableAccess` ARN |
| ASG can't reach targets | Medium | Low | Verify SG allows HTTP 80 from `0.0.0.0/0`; check ALB health status |

---

### 8. Expected Outcomes

- A **working, full-stack serverless app** (CloudNote) running live on AWS Free Tier with a browser demo.
- Deepened understanding of IAM least-privilege, DynamoDB, Lambda + API Gateway, S3 static hosting, CloudWatch and CloudTrail.
- A second **bonus demo**: ALB load-balancing across 2 EC2 instances in a custom VPC.
- A complete set of screenshots/video/report material for the internship deliverable and self-evaluation.