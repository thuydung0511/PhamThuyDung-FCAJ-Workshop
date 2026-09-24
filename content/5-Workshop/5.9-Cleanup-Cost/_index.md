---
title: "Resource cleanup and cost control"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Goal

- Delete all Part B resources right after testing so they do not keep generating charges.
- Keep the cost of the whole workshop at zero within the Free Tier.

## Steps

### 1. Part B cleanup

Delete in dependency order (dependent resources first):

1. **Auto Scaling Group** `cloudnote-asg`.
2. **Load Balancer** `cloudnote-alb`.
3. **Target Group** `cloudnote-tg`.
4. **Launch Template** `cloudnote-web-template`.
5. **VPC** `cloudnote-vpc`, with its subnets, route table, Internet Gateway and security group `cloudnote-web-sg`.
6. **Key Pair** `cloudnote-key`.

Then check each item again to confirm that no Part B resources remain.

### 2. Cost control

- **Zero-Spend Budget:** email alert as soon as any cost appears. IAM users cannot access Billing by default, so I signed in as root exactly once to create the budget, then signed out.
- **No NAT Gateway** in Part B; EC2 uses public subnets with auto-assign public IPv4 instead.
- **No upgrade to the Paid plan** when Bedrock, Comprehend and Translate were blocked; switched to Amazon Polly ([5.6](../5.6-polly-read-note/)).
- DynamoDB runs in **On-demand** mode; Polly receives at most 3000 characters per request (`text[:3000]`).

### 3. CloudNote resources kept

Part A is still running for the demo: IAM user/group/role, DynamoDB table `Notes`, Lambda `notes-api`, HTTP API `notes-http-api`, the static website S3 bucket and the CloudTrail log bucket, Cognito User Pool, CloudWatch dashboard and alarm, SNS topic `cloudnote-alerts`, trail `cloudnote-audit-trail`, Zero-Spend Budget.

## Issues and fixes

| Issue | Fix |
|-------|-----|
| The IAM user could not view Billing | AWS blocks IAM users from Billing by default → signed in as root once to create the Zero-Spend Budget, then signed out |

## Results

- All Part B resources were deleted; no ongoing charges.
- The Zero-Spend Budget is active.
