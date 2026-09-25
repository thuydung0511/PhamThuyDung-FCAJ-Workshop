---
title: "Week 5 Worklog"
date: 2026-09-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:

* Continue building the **CloudNote** capstone project on AWS (started in Week 4 with A0–A2), following the architecture designed in Week 4.
* Add user authentication, per-user data isolation and one AI feature.
* Do Part B separately as a practice exercise: VPC, ALB, Auto Scaling, then clean up the resources.
* Publish the internship report website on GitHub Pages.

**Period:** 20/09/2026 – 27/09/2026 · Region: ap-southeast-1 (Singapore)

### Tasks carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | ---------- | --------------- | ------------------ |
| Sun | - Completed A3–A8 (continuing the work started on 19/09, see [Week 4](../1.4-week4/)): Lambda CRUD, API Gateway, API tests, S3 frontend, end-to-end test, CloudWatch dashboard, alarm and SNS <br> - Part B preparation: IAM group with VPC/EC2/ELB permissions | 20/09/2026 | 20/09/2026 | |
| Mon | - Part B: 2-AZ VPC (B1), Launch Template and security group (B2) | 21/09/2026 | 21/09/2026 | |
| Tue | - CloudTrail (A9) <br> - Created the Cognito User Pool <br> - Part B: Target Group, ALB, Auto Scaling, load-balancing test (B3–B5), cleanup | 22/09/2026 | 22/09/2026 | |
| Wed | - Continued authentication: JWT Authorizer, sign-up/sign-in on the frontend, Edit button, filtering data by `userId` <br> - Tried Bedrock and Comprehend (blocked) <br> - Pushed the report website to GitHub | 23/09/2026 | 23/09/2026 | |
| Thu | - Fixed the GitHub Pages 404 error <br> - Tried Translate and Polly; built the "Read note" feature with Amazon Polly <br> - Opened an AWS Support case requesting Bedrock access | 24/09/2026 | 24/09/2026 | |

### Implementation details

#### 1. Serverless backend (A3–A5)

**Steps**
* A0–A2 (IAM user, DynamoDB table `Notes`, role `LambdaNotesExecutionRole`) were done on 19/09, see [Week 4](../1.4-week4/).
* **A3 – Lambda:** function `notes-api` (Python 3.12) handling GET/POST/PUT/DELETE; ran a Test event and checked the item in DynamoDB.
* **A4 – API Gateway:** HTTP API `notes-http-api` with 4 routes `GET /notes`, `POST /notes`, `PUT /notes/{noteId}`, `DELETE /notes/{noteId}`; CORS enabled.
* **A5 – API tests:** tested all 4 methods with PowerShell (`Invoke-RestMethod`) and the preflight with an OPTIONS request.

**Issues and fixes**
* CloudShell reported missing permissions → attached `AWSCloudShellFullAccess`. CloudShell then reported the account was still being verified (up to 2 days) → tested with PowerShell on my own machine instead.
* `GET /notes` returned notes in the wrong order because `Scan()` does not guarantee ordering → sorted by `createdAt` descending in Lambda.

**Results**
* All 4 CRUD routes work; data is created, updated and deleted correctly in DynamoDB; PUT keeps the original `createdAt`.
* The OPTIONS preflight returns 204 with all CORS headers.

#### 2. Frontend, monitoring, auditing (A6–A9)

**Steps**
* **A6 – S3:** bucket `cloudnote-app-0205568-2026` with Static website hosting and a Bucket Policy allowing public read; `index.html` written in plain HTML/CSS/JS, with an HTML-escaping function to prevent XSS.
* **A7 – End-to-end test:** added/deleted notes on the website, reloaded the page and compared with DynamoDB.
* **A8 – CloudWatch:** dashboard `CloudNote-Dashboard` (Lambda: Duration, Errors, Invocations; DynamoDB: Consumed Read/Write Capacity), alarm `notes-api-error-alarm` (Lambda Errors > 0 within 5 minutes), SNS topic `cloudnote-alerts` sending email.
* **A9 – CloudTrail:** trail `cloudnote-audit-trail` (multi-region), recording Read + Write management events.
* **Cost:** created a Zero-Spend Budget.

**Issues and fixes**
* The alarm stayed in "Insufficient data" because of missing datapoints → set "Treat missing data as good"; the alarm moved to OK.
* The IAM user could not view Billing (blocked by default) → signed in as root once to create the budget, then signed out.

**Results**
* Adding, deleting and reloading on the website all work; the newest note is shown first.
* The dashboard shows data; the alarm is OK; the SNS email is confirmed; CloudTrail is Logging.

#### 3. Part B: VPC, EC2, ALB, Auto Scaling (separate exercise)

**Steps**
* Granted VPC/EC2/ELB permissions through IAM group `cloudnote-network-group` (the user had reached the limit of 10 directly attached managed policies).
* **B1:** VPC `cloudnote-vpc` (10.0.0.0/16), 2 public subnets in 2 AZs, Internet Gateway `cloudnote-igw`, route table `cloudnote-rtb-public`, no NAT Gateway; auto-assign public IPv4 enabled on the subnets.
* **B2:** Launch Template `cloudnote-web-template` (Amazon Linux 2023, t3.micro), security group `cloudnote-web-sg` (HTTP 80 open, SSH 22 only from my IP); User Data installs Apache and reads the Instance ID via IMDSv2.
* **B3:** Target Group `cloudnote-tg`, ALB `cloudnote-alb` (Internet-facing, HTTP:80 listener).
* **B4:** Auto Scaling Group `cloudnote-asg` (Desired 2 / Min 1 / Max 2), EC2 and ELB health checks enabled.
* **B5:** opened the ALB DNS name and reloaded several times to check load balancing.
* **Cleanup:** deleted in order ASG → ALB → Target Group → Launch Template → VPC → Key Pair.

**Issues and fixes**
* The VPC wizard defaulted to 0 public / 2 private subnets → changed to 2 public / 0 private.
* The ALB also had the `default` security group selected → removed it, kept only `cloudnote-web-sg`.
* The ASG was set to "No load balancer" → changed to "Attach to an existing load balancer" with Target Group `cloudnote-tg`.
* The browser timed out on the ALB DNS name → `nslookup` and `curl` showed the infrastructure was fine; the cause was browser cache/extensions, and it worked in a private window.

**Results**
* 2 Healthy instances; the ALB returns HTTP 200; the Instance ID alternates between the 2 instances on reload.
* All Part B resources were deleted.

#### 4. User authentication and data isolation

**Steps**
* **Cognito:** User Pool with email sign-in and self-registration; App Client of type Single-page application (no client secret).
* **API Gateway:** JWT Authorizer on `notes-http-api` (Issuer = User Pool, Audience = App Client), attached to all 4 routes.
* **Frontend:** Sign up / Confirm email / Sign in / Sign out forms (library `amazon-cognito-identity-js`); an `authFetch()` function adds `Authorization: Bearer <token>` to every request; added an **Edit** button using the existing PUT route.
* **Lambda `notes-api`:** reads `sub` from the JWT claims as `userId`; GET returns only the caller's notes; POST stores `userId`; PUT/DELETE use a `ConditionExpression` to check ownership and return 403 if it does not match.

**Issues and fixes**
* `AccessDeniedException` when opening Cognito → attached `AmazonCognitoPowerUser` to the user.
* The preflight was blocked when sending the token because CORS did not allow the `Authorization` header → added `Authorization` to the CORS configuration.
* Error 500 when creating a note because DynamoDB does not accept `float` for `createdAt` → converted to `Decimal`.
* Old notes disappeared after filtering by `userId` because they had no such field → expected behavior; they were test data.

**Results**
* Sign up, email confirmation, sign in and sign out work.
* Calling the API without a token → **401 Unauthorized**; with a valid token → Add/Edit/Delete work normally.
* Tested with 2 accounts: each account sees only its own notes.

#### 5. AI feature: "Read note" with Amazon Polly

**Steps**
* Tried Amazon Bedrock and Amazon Comprehend, then Amazon Translate: all three were blocked because the account is on the Free account plan. I did not upgrade to the Paid plan to avoid costs.
* Tried Amazon Polly in the Console: English worked well (Standard engine, voice Joanna); Vietnamese pronunciation was poor because there is no vi-VN voice → chose Polly and demoed with English notes.
* **Permissions:** inline policy `PollySynthesizeSpeech` on `LambdaNotesExecutionRole`, containing only `polly:SynthesizeSpeech`.
* **API Gateway:** route `POST /notes/{noteId}/speak` with the JWT Authorizer and a new Lambda integration.
* **Lambda:** function `speak_note` checks that the note belongs to the caller before calling Polly and returns MP3; routing by `routeKey`; timeout increased from 3 to 10 seconds.
* **Frontend:** a "🔊 Read" button calls the API through `authFetch` and plays the MP3.
* Opened an AWS Support case requesting Bedrock access (Bedrock Allowlisting); waiting for a response.

**Issues and fixes**
* Missing `polly:DescribeVoices` and `support:DescribeSupportLevel` permissions in the Console → added inline policy `ConsoleTestPolly` and `AWSSupportAccess` to group `cloudnote-network-group`.
* The API was not visible in API Gateway because the Console was in us-east-1 → switched back to ap-southeast-1.
* A `/speak` request could be handled by the "create note" POST branch → routed by `routeKey` and placed the `/speak` branch first.
* The first "another user's note" test was invalid (sent the placeholder text instead of a real noteId) → took a real noteId from account A and retested with account B.

**Results**
* Clicking "🔊 Read" while signed in: preflight 204, request 200, MP3 plays.
* Calling `/speak` without a token: **401**.
* Account B calling `/speak` on account A's note: **403**, and Lambda does not call Polly.
* Non-existent noteId: **403**, without revealing whether the note exists.

#### 6. Internship report website

* Pointed `git remote` to my own repository and pushed to `main`; the GitHub Actions workflow builds the Hugo site and deploys it.
* **404 error although the workflow succeeded:** the workflow pushes the site to the `gh-pages` branch, while Settings → Pages had Source set to "GitHub Actions" → changed to "Deploy from a branch", branch `gh-pages`. The website is now reachable.

### Final CloudNote architecture

The browser loads the static website from Amazon S3. The user signs in with Amazon Cognito and receives a JWT. Every API call to API Gateway carries the JWT; the JWT Authorizer validates the token before forwarding the request to the Lambda function `notes-api`. Lambda reads and writes the DynamoDB table `Notes`, always filtering by the caller's `userId`, and calls Amazon Polly for the "Read note" feature. Amazon CloudWatch monitors and alerts on errors, and AWS CloudTrail records API calls. Part B (VPC, ALB, Auto Scaling) was a separate exercise, has been cleaned up and is not part of the CloudNote architecture.

### Not done

* Amazon ECS / Docker: not practiced yet; I prioritized completing the capstone project.

### Week 5 Achievements:

* CloudNote runs end-to-end on S3, API Gateway, Lambda and DynamoDB, with Cognito sign-in and per-user data isolation (401/403 as designed).
* Added a "Read note" feature with Amazon Polly, including ownership checks.
* Set up monitoring (CloudWatch dashboard, alarm, SNS) and auditing (CloudTrail); cost control with a Zero-Spend Budget.
* Completed the VPC/ALB/Auto Scaling exercise and deleted all its resources after testing.
* The report website is live on GitHub Pages.
