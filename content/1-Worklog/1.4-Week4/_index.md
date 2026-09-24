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

**Period:** 12/09/2026 – 19/09/2026

### Tasks carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | ---------- | --------------- | ------------------ |
| Sat | Attended a session at the AWS Hanoi office | 12/09/2026 | 12/09/2026 | |
| Sat | Attended a session at the AWS Hanoi office | 19/09/2026 | 19/09/2026 | |

Self-study during the week: Lambda, API Gateway, DynamoDB, CloudWatch and CloudTrail; CloudNote architecture design.

### Week 4 Achievements:

* Understood how Lambda functions are triggered, how they get permissions through an execution role and how they write logs to CloudWatch.
* Understood the difference between CloudWatch (metrics, logs, alarms) and CloudTrail (recording API calls for auditing).
* Designed the CloudNote architecture:
  * Static frontend hosted on Amazon S3.
  * REST-style CRUD API on API Gateway (HTTP API) calling a Lambda function.
  * Notes stored in a DynamoDB table.
  * Monitoring with CloudWatch and auditing with CloudTrail.
* Listed the resources to create and the order of implementation for Week 5.
