---
title: "Week 6 Worklog"
date: 2026-09-05
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

* Begin the capstone project **CloudNote** — a serverless notes application on AWS Free Tier.
* Build the serverless core: IAM practice user, DynamoDB table, and Lambda CRUD backend.

**Period:** 05/09/2026 – 11/09/2026

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | ---------- | --------------- | ------------------ |
| 2 | - Attend project kickoff: CloudNote architecture overview, deliverables | 05/09/2026 | 05/09/2026 | FCAJ project brief |
| 3 | - Create IAM practice user `cloudnote-dev` (never use root) | 06/09/2026 | 06/09/2026 | |
| 4 | - Create DynamoDB table `Notes` (partition key `noteId`, on-demand capacity) | 07/09/2026 | 07/09/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Create Lambda execution role `LambdaNotesExecutionRole` with `NotesTableAccess` | 08/09/2026 | 08/09/2026 | |
| 6 | - Implement `notes-api` Lambda (Python 3.12) handling GET/POST/PUT/DELETE | 09/09/2026 | 09/09/2026 | |

### Week 6 Achievements:

* Kicked off the **CloudNote** capstone — a full-stack serverless notes app: S3 frontend + API Gateway + Lambda + DynamoDB.
* Created the `cloudnote-dev` IAM practice user following the least-privilege and no-root principles.
* Provisioned the `Notes` DynamoDB table with on-demand capacity and partition key `noteId`.
* Built and deployed the `notes-api` Lambda performing CRUD operations against DynamoDB.
* Established the serverless foundation that will be exposed through API Gateway in Week 7.