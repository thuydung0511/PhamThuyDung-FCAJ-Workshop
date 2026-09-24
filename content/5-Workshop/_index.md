---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Workshop: CloudNote — a serverless notes application on AWS

This workshop documents the steps I carried out to build the **CloudNote** capstone project: a personal notes web application with sign-in, running on AWS serverless services within the Free Tier. The static frontend is stored on **Amazon S3**, users sign in with **Amazon Cognito**, the CRUD API runs on **API Gateway + AWS Lambda**, data is stored in **Amazon DynamoDB**, the "Read note" feature uses **Amazon Polly**, monitoring uses **Amazon CloudWatch** and auditing uses **AWS CloudTrail**. Part B is a separate exercise on **VPC, EC2, ALB and Auto Scaling**, cleaned up after testing.

**Region:** `ap-southeast-1` (Singapore)

#### Contents

1. [Workshop overview](5.1-workshop-overview/)
2. [IAM: user, role and group](5.2-iam/)
3. [Backend: DynamoDB, Lambda, API Gateway](5.3-backend/)
4. [Frontend on S3](5.4-frontend/)
5. [User authentication with Cognito](5.5-authentication/)
6. ["Read note" feature with Amazon Polly](5.6-polly-read-note/)
7. [Monitoring and auditing: CloudWatch, CloudTrail](5.7-monitoring/)
8. [Part B: VPC, ALB, Auto Scaling](5.8-part-b-web-tier/)
9. [Resource cleanup and cost control](5.9-cleanup-cost/)
