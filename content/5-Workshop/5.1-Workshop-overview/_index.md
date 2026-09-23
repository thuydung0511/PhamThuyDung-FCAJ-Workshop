---
title: "Workshop overview"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Purpose

This workshop documents the AWS implementation steps for my capstone project **CloudNote** — a **serverless notes application** on AWS Free Tier. Users open a static website (hosted on **S3**), then add, edit, view and delete notes through the web UI; all data is stored and processed serverless (no servers to manage).

| Part | Components | Workshop coverage |
|------|------------|-------------------|
| **A** | S3 frontend, API Gateway, Lambda, DynamoDB | IAM user, `Notes` DynamoDB table, `notes-api` Lambda, HTTP API + CORS, S3 static hosting, CloudWatch, CloudTrail |
| **B** | VPC, EC2, ALB, Auto Scaling | Custom VPC, launch template, target group + ALB, ASG, load-balancing test |

Part A is the core capstone deliverable; Part B is a bonus horizontal-scaling web tier.

## Prerequisites

- AWS account (new account recommended to benefit from the 12-month Free Tier) with admin access in `ap-southeast-1`
- Region set to `ap-southeast-1`
- A browser is enough — the whole workshop runs on the **AWS Console** (AWS CLI optional)
- CloudNote source repository: *will be updated soon*

## Resource naming reference

| Resource | Name / pattern |
|----------|----------------|
| IAM practice user | `cloudnote-dev` |
| DynamoDB table | `Notes` (partition key `noteId`) |
| Lambda function | `notes-api` (Python 3.12) |
| Lambda execution role | `LambdaNotesExecutionRole` (+ inline `NotesTableAccess`) |
| API Gateway (HTTP API) | `notes-http-api` |
| S3 bucket | `cloudnote-app-0205568-2026` |
| CloudWatch dashboard / alarm | `CloudNote-Dashboard` / `notes-api-error-alarm` |
| CloudTrail trail | `cloudnote-audit-trail` |
| VPC | `cloudnote-vpc` |
| Launch template | `cloudnote-web-template` |
| Target group / ALB / ASG | `cloudnote-tg` / `cloudnote-alb` / `cloudnote-asg` |

## Workshop order

Complete **Part A** first (A0 → A9) to get a working full-stack serverless app, then **Part B** (B1 → B5) for the bonus web tier. Follow the documented **cleanup** section right after finishing Part B to avoid ALB charges.

## Verification checklist

After all sections:

- [ ] DynamoDB table `Notes` is `Active` and accepts items
- [ ] `notes-api` Lambda returns 201 on POST test from the console
- [ ] API Gateway routes (`GET/POST/PUT/DELETE /notes`) invoke the Lambda; CORS enabled
- [ ] S3 website URL loads the CloudNote UI; add/delete notes end-to-end
- [ ] CloudWatch dashboard shows real invocations/errors; alarm is OK
- [ ] CloudTrail trail status is `Logging`
- [ ] (Part B) ALB DNS alternates between the two EC2 instances behind the ASG