---
title: "Proposal"
date: 2026-08-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# CloudNote — Serverless Notes Application on AWS

## A Full-Stack, Cost-Optimized Serverless Reference Application on AWS

---

### 1. Executive Summary

**CloudNote** is a full-stack serverless notes application built as the Capstone Project for the *First Cloud AI Journey (FCAJ)* internship at Amazon Web Services Vietnam. Users open a static website (hosted on **S3**), sign up and sign in with **Amazon Cognito**, then view, create, edit and delete their notes in the browser, and listen to a note read aloud by **Amazon Polly**. Each user can only see and change their own notes. All data is stored and processed by serverless AWS services — no servers to manage.

The project follows a core principle: *run everything serverless, understand each step, and keep costs minimal*. The core covers IAM, DynamoDB, Lambda, API Gateway, S3 static hosting, Cognito authentication, Polly, CloudWatch monitoring with SNS email alerts and CloudTrail auditing, while a bonus web tier (Part B) demonstrates horizontal scaling with VPC, EC2 Launch Templates, an Application Load Balancer and Auto Scaling.

---

### 2. Problem Statement

#### What's the Problem?

Building and demonstrating a "real" application on the cloud usually requires understanding many moving parts: identity and access, a data store, a compute layer, an API surface, frontend hosting, monitoring and auditing. Beginners often either stay with toy examples or jump straight to containers/Kubernetes without mastering the fundamentals.

#### The Solution

CloudNote is deliberately simple in scope but complete in its lifecycle. One **Lambda** function exposes CRUD operations and a "Read note" operation through an **API Gateway HTTP API**, backed by a **DynamoDB** table. A static frontend on **S3** calls the API. **Cognito** handles sign-up and sign-in, and a **JWT Authorizer** on API Gateway only lets requests with a valid token through. **Polly** turns notes into speech. **CloudWatch**, **SNS** and **CloudTrail** provide observability, alerting and auditability. A bonus **VPC + EC2 + ALB + Auto Scaling** web tier adds the classic scale-out web serving pattern.

Benefits:

- **100% Serverless Core**: No EC2 to manage for the main application; Lambda and DynamoDB scale automatically.
- **Low cost**: The account is on the Free account plan and runs on credits; the whole project used $3.84 of the $200 credit.
- **Fundamentals Covered**: IAM least-privilege, NoSQL data modeling, API design, CORS, JWT authentication, per-user data isolation, monitoring, auditing and horizontal scaling.
- **Reusable Blueprint**: The same shape applies to any tiny API app (to-do list, surveys, personal blog backend).

---

### 3. Solution Architecture

The main flow is fully serverless; the bonus lab adds a traditional scale-out web tier.

![CloudNote Serverless Architecture](/images/5-Workshop/architecture-cloudnote.png)

#### Main Flow (Part A — Serverless Core)

1. The user opens the static website URL on **S3** and receives `index.html` (HTML, CSS and JavaScript in a single file).
2. The user signs up / signs in with the **Cognito User Pool** and receives a JWT.
3. The browser calls the **API Gateway HTTP API** (`/notes`) with the header `Authorization: Bearer <JWT>`.
4. The **JWT Authorizer** validates the token; valid requests are forwarded to the **Lambda** function `notes-api` (Python 3.12), requests without a valid token get **401**.
5. **Lambda** takes the `sub` claim as `userId` and uses its IAM Role (`LambdaNotesExecutionRole`) to read/write the **DynamoDB** table `Notes`, always filtered by `userId`; editing, deleting or reading another user's note returns **403**.
6. For "Read note", Lambda calls **Amazon Polly** (Standard engine, voice Joanna) and returns MP3 for the browser to play.
7. **CloudWatch** records logs and metrics, and the alarm sends an email through **SNS** when Lambda has errors; **CloudTrail** records API calls in the account for auditing.

#### Bonus Flow (Part B — Web Tier)

- A custom **VPC** with 2 public subnets across 2 Availability Zones.
- An **EC2 Launch Template** (Amazon Linux 2023, t3.micro) with a User Data script that serves an instance-ID page.
- A **Target Group + Application Load Balancer (ALB)** and an **Auto Scaling Group** that keep 1–2 web instances running; refreshing the ALB DNS name alternates the displayed Instance ID.
- All Part B resources are deleted after testing.

#### AWS Services Used

- **IAM**: practice user `cloudnote-dev`; group `cloudnote-network-group`; execution role `LambdaNotesExecutionRole` + inline policies `NotesTableAccess` and `PollySynthesizeSpeech`.
- **Amazon DynamoDB**: table `Notes` (partition key `noteId`, on-demand capacity); each note stores the owner's `userId`.
- **AWS Lambda**: `notes-api` — a single function handling GET/POST/PUT/DELETE and "Read note", filtering data by `userId`.
- **Amazon API Gateway**: HTTP API `notes-http-api` with 5 routes — `GET/POST /notes`, `PUT/DELETE /notes/{noteId}` and `POST /notes/{noteId}/speak` — all protected by a JWT Authorizer.
- **Amazon S3**: bucket `cloudnote-app-0205568-2026` — static website hosting + public read bucket policy.
- **Amazon Cognito**: User Pool (email sign-in) and an App Client of type Single-page application.
- **Amazon Polly**: Standard engine, voice Joanna, MP3 output.
- **Amazon CloudWatch**: dashboard `CloudNote-Dashboard`; alarm `notes-api-error-alarm`.
- **Amazon SNS**: topic `cloudnote-alerts` sending alarm emails.
- **AWS CloudTrail**: trail `cloudnote-audit-trail` (multi-region, management events).
- **VPC / EC2 / ELB / Auto Scaling** (Part B): `cloudnote-vpc`, `cloudnote-web-template`, `cloudnote-tg`, `cloudnote-alb`, `cloudnote-asg`.

---

### 4. Technical Implementation

#### Implementation Steps (Part A)

1. **A0 — IAM user**: create `cloudnote-dev` with console access and attach 7 managed FullAccess policies for practice (S3, DynamoDB, Lambda, API Gateway, IAM, CloudWatch, CloudTrail); use this user instead of root.
2. **A1 — DynamoDB**: create table `Notes`, partition key `noteId` (String), **On-demand** capacity.
3. **A2 — IAM Role**: create `LambdaNotesExecutionRole` with `AWSLambdaBasicExecutionRole` + inline `NotesTableAccess` (DynamoDB CRUD scoped to the `Notes` table ARN).
4. **A3 — Lambda**: create `notes-api` (Python 3.12), attach the role, implement CRUD, run a Test event and check the item in DynamoDB.
5. **A4 — API Gateway**: build HTTP API `notes-http-api`, add 4 CRUD routes to `notes-api`, enable CORS, note the Invoke URL.
6. **A5 — API test**: test all 4 methods and the CORS preflight with PowerShell (`Invoke-RestMethod`); CloudShell could not be used because the new account was still being verified.
7. **A6 — S3 hosting**: create bucket `cloudnote-app-0205568-2026`, enable static website hosting, attach public-read bucket policy, upload `index.html` (frontend calls the Invoke URL).
8. **A7 — E2E test**: open the website endpoint, create and delete notes in the browser, reload and compare with DynamoDB.
9. **A8 — CloudWatch**: build dashboard with Lambda `Invocations/Errors/Duration` + DynamoDB `Consumed*Capacity`, create alarm on Lambda Errors, send alerts by email through SNS topic `cloudnote-alerts`.
10. **A9 — CloudTrail**: create trail `cloudnote-audit-trail` (multi-region, management events, new S3 bucket for logs).
11. **Authentication**: create the Cognito User Pool and App Client, attach a JWT Authorizer to all routes, add sign-up/sign-in to the frontend, and filter data by `userId` in Lambda (PUT/DELETE check ownership).
12. **"Read note"**: add inline policy `PollySynthesizeSpeech` (only `polly:SynthesizeSpeech`) to the Lambda role, add route `POST /notes/{noteId}/speak` (5 routes in total), call Polly from Lambda after checking the note belongs to the caller, and add a "🔊 Read" button to the frontend.

#### Implementation Steps (Part B — optional)

B1 custom VPC (2 AZ, 2 public subnets, no NAT) → B2 launch template `cloudnote-web-template` (t3.micro) → B3 target group `cloudnote-tg` + ALB `cloudnote-alb` → B4 ASG `cloudnote-asg` (desired 2, min 1, max 2, ELB health checks) → B5 verify load balancing alternates instance IDs → cleanup of all Part B resources.

B6 (advanced, optional) Docker + ECS Fargate: **not done**.

#### Technical Requirements

- AWS Console only (no CLI required); region **ap-southeast-1 (Singapore)**.
- The API is tested with PowerShell on a local machine.

#### Security Measures Implemented

- **MFA** (Authenticator app) is enabled on the root account; the root account has no access keys.
- Daily work uses IAM user `cloudnote-dev`, not root.
- The Lambda role only has the permissions it needs (least privilege): logs, 6 actions on the `Notes` table and `polly:SynthesizeSpeech`.
- Every API route requires a valid Cognito JWT; Lambda checks note ownership by `userId`.
- The frontend escapes note content before inserting it into HTML (XSS prevention) and keeps the token in memory only, not in `localStorage`.

---

### 5. Timeline & Milestones

- **Week 1 (01/08 – 14/08)**: AWS global architecture overview; created the AWS account; studied Lab 000001 (AWS Free Tier).
- **Week 2 (15/08 – 28/08)**: Core services EC2, S3, IAM; Lab 000004; completed the 5 "Explore AWS" tasks of Lab 000001.
- **Week 3 (29/08 – 11/09)**: AWS networking (VPC, subnets, Internet Gateway); chose the CloudNote idea.
- **Week 4 (12/09 – 19/09)**: Lambda, serverless, CloudWatch, CloudTrail; CloudNote architecture design; started Part A (A0–A2).
- **Week 5 (20/09 – 27/09)**: Completed Part A (A3–A9), Part B web tier and cleanup, Cognito authentication, "Read note" with Polly, final report website.

---

### 6. Budget Estimation & Cost Optimization

The AWS account is on the **Free account plan** and runs on credits: $100 when the account was created and $100 more for completing the "Explore AWS" tasks, $200 in total. Region `ap-southeast-1`.

| Service | Workshop usage | Cost |
|---|---|---|
| S3 | 1 website bucket + 1 CloudTrail log bucket | Covered by credits |
| DynamoDB (on-demand) | 1 table, a few items | Covered by credits |
| AWS Lambda | 1 function, low number of calls | Covered by credits |
| API Gateway (HTTP API) | 1 API, 5 routes | Covered by credits |
| Cognito | 1 User Pool, a few users | Covered by credits |
| Polly | Standard engine, at most 3000 characters per request | Covered by credits |
| CloudWatch + SNS | 1 dashboard, 1 alarm, 1 email topic | Covered by credits |
| CloudTrail | 1 trail | Covered by credits |
| EC2 + ALB (Part B) | 2 t3.micro instances + 1 ALB, deleted after testing | Covered by credits |
| **Actual total** | | **$3.84 of the $200 credit used; $0.00 paid** |

Budget controls: a **Zero-Spend Budget** in AWS Billing, no NAT Gateway in Part B, Part B deleted right after testing, and no upgrade to the Paid plan.

---

### 7. Risk Assessment

| Risk Item | Impact | Probability | Mitigation Strategy |
| --- | --- | --- | --- |
| Unexpected charges (ALB) | Medium | Medium | Zero-spend budget; delete Part B right after testing; follow the cleanup order in the workshop |
| CORS errors in the browser | Medium | High | Configure CORS on API Gateway (including the `Authorization` header); verify with the frontend |
| Lambda 500 errors | Medium | Medium | Check CloudWatch logs (`/aws/lambda/notes-api`) and verify `NotesTableAccess` ARN |
| One user reading or changing another user's notes | High | Medium | JWT Authorizer on all routes; Lambda filters by `userId` and checks ownership (403) |
| AI services blocked on the Free account plan | Medium | High | Use Amazon Polly, which is available on the Free account plan |
| ASG can't reach targets | Medium | Low | Verify SG allows HTTP 80 from `0.0.0.0/0`; check ALB health status |

---

### 8. Expected Outcomes

- A **working, full-stack serverless app** (CloudNote) running live on AWS with a browser demo, Cognito sign-in, per-user data and a "Read note" feature.
- Deepened understanding of IAM least-privilege, DynamoDB, Lambda + API Gateway, S3 static hosting, Cognito, Polly, CloudWatch and CloudTrail.
- A second **bonus demo**: ALB load-balancing across 2 EC2 instances in a custom VPC.
- A complete set of screenshots and report material for the internship deliverable and self-evaluation.
