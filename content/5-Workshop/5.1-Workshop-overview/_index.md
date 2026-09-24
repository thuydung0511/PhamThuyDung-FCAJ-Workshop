---
title: "Workshop overview"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Goal

Build **CloudNote**, a personal notes web application on AWS: users sign up, sign in, then add, edit, delete and listen to their notes. Each user can only see and change their own notes. Everything runs on serverless services within the Free Tier.

## Architecture

The browser loads the static website (HTML/CSS/JS) from **Amazon S3**. The user signs in with **Amazon Cognito** and receives a JWT. Every call to **API Gateway** (HTTP API) carries the JWT; the **JWT Authorizer** validates the token before forwarding the request to the **AWS Lambda** function `notes-api`. Lambda reads and writes the **Amazon DynamoDB** table `Notes`, always filtering by the caller's `userId`, and calls **Amazon Polly** to generate speech for the "Read note" feature. **Amazon CloudWatch** monitors and alerts on errors, and **AWS CloudTrail** records API calls in the account.

Part B (VPC, EC2, ALB, Auto Scaling) is a separate exercise on a horizontally scalable web tier. It was cleaned up after testing and is not part of the CloudNote architecture.

> Architecture diagram: will be updated.

## Prerequisites

- An AWS account (Free account plan), Region `ap-southeast-1` (Singapore).
- Work in the AWS Console; test the API with PowerShell on a local machine.
- Do not use the root account for daily work (see [5.2](../5.2-iam/)).

## Resource names

| Service | Resources |
|---------|-----------|
| IAM | User `cloudnote-dev`; group `cloudnote-network-group`; role `LambdaNotesExecutionRole` (inline policies `NotesTableAccess`, `PollySynthesizeSpeech`) |
| DynamoDB | Table `Notes` (partition key `noteId`) |
| Lambda | `notes-api` (Python 3.12) |
| API Gateway | HTTP API `notes-http-api` |
| S3 | Static website bucket `cloudnote-app-0205568-2026` |
| Cognito | 1 User Pool (email sign-in), 1 App Client (SPA) |
| Polly | Standard engine, voice Joanna, MP3 output |
| CloudWatch | Dashboard `CloudNote-Dashboard`, alarm `notes-api-error-alarm` |
| SNS | Topic `cloudnote-alerts` |
| CloudTrail | Trail `cloudnote-audit-trail` |
| Part B (deleted) | `cloudnote-vpc`, `cloudnote-web-template`, `cloudnote-web-sg`, `cloudnote-key`, `cloudnote-tg`, `cloudnote-alb`, `cloudnote-asg` |

## Order of work

1. [IAM](../5.2-iam/): working user, Lambda role, group when the policy limit is reached.
2. [Backend](../5.3-backend/): DynamoDB, Lambda, API Gateway, API tests.
3. [Frontend](../5.4-frontend/): S3 static website, end-to-end test.
4. [Authentication](../5.5-authentication/): Cognito, JWT Authorizer, per-user data isolation.
5. ["Read note"](../5.6-polly-read-note/): Amazon Polly.
6. [Monitoring](../5.7-monitoring/): CloudWatch, SNS, CloudTrail.
7. [Part B](../5.8-part-b-web-tier/): VPC, ALB, Auto Scaling.
8. [Cleanup and cost](../5.9-cleanup-cost/).

Real values (account ID, User Pool ID, App Client ID, Invoke URL, ALB DNS name) are replaced with placeholders such as `<ACCOUNT_ID>`, `<USER_POOL_ID>`, `<APP_CLIENT_ID>`, `<API_URL>`, `<REGION>`, `<ALB_DNS_NAME>`.
