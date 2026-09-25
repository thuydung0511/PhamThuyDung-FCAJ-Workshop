---
title: "Week 4 Worklog"
date: 2026-09-12
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

* Learn AWS Lambda and the serverless model.
* Learn monitoring and auditing with Amazon CloudWatch and AWS CloudTrail.
* Design the architecture of CloudNote.
* Start building CloudNote: IAM user, DynamoDB table and Lambda execution role (A0–A2).

**Period:** 12/09/2026 – 19/09/2026

### Tasks carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | ---------- | --------------- | ------------------ |
| Sat | Attended a session at the AWS Hanoi office | 12/09/2026 | 12/09/2026 | |
| Sat | Attended a session at the AWS Hanoi office | 19/09/2026 | 19/09/2026 | |
| Sat | Started the CloudNote hands-on work (A0–A2): IAM user `cloudnote-dev`, DynamoDB table `Notes`, IAM role `LambdaNotesExecutionRole` | 19/09/2026 | 19/09/2026 | |

Self-study during the week: Lambda, API Gateway, DynamoDB, CloudWatch and CloudTrail; CloudNote architecture design.

### Week 4 Achievements:

* Understood how Lambda functions are triggered, how they get permissions through an execution role and how they write logs to CloudWatch.
* Understood the difference between CloudWatch (metrics, logs, alarms) and CloudTrail (recording API calls for auditing).
* Designed the CloudNote architecture:
  * Static frontend hosted on Amazon S3.
  * REST-style CRUD API on API Gateway (HTTP API) calling a Lambda function.
  * Notes stored in a DynamoDB table.
  * Monitoring with CloudWatch and auditing with CloudTrail.
* Listed the resources to create and the order of implementation.
* Started the hands-on work on 19/09/2026:
  * **A0 – IAM:** created IAM user `cloudnote-dev` with 7 managed policies (S3, DynamoDB, Lambda, API Gateway, IAM, CloudWatch, CloudTrail); worked with this user from then on instead of root.
  * **A1 – DynamoDB:** table `Notes`, partition key `noteId` (String), On-demand mode.
  * **A2 – IAM Role:** `LambdaNotesExecutionRole` with `AWSLambdaBasicExecutionRole` and inline policy `NotesTableAccess`, allowing only Put/Get/Update/Delete/Scan/Query on the `Notes` table.
* Continued with A3–A8 over 19–20/09, see [Week 5](../1.5-week5/).
