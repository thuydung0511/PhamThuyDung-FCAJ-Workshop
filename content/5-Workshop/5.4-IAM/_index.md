---
title: "IAM"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Goal

Configure IAM so the **CloudNote** serverless app has least-privilege access: the `notes-api` Lambda can read/write the `Notes` table and log to CloudWatch; the practice user never uses the root account.

## Step 1 — IAM practice user (never use root)

1. **IAM → Users → Create user**. User name: `cloudnote-dev`.
2. Choose **Attach policies directly** and attach the practice policies (wide scope for learning, not for production):
   - `AmazonS3FullAccess`, `AmazonDynamoDBFullAccess`, `AWSLambda_FullAccess`
   - `AmazonAPIGatewayAdministrator`, `IAMFullAccess`, `CloudWatchFullAccess`, `AWSCloudTrail_FullAccess`
   - (Part B only) `AmazonVPCFullAccess`, `AmazonEC2FullAccess`, `ElasticLoadBalancingFullAccess`
3. Enable **Console access** for this user and log out of root.
4. Use `cloudnote-dev` for the entire workshop.

## Step 2 — Lambda execution role

1. **IAM → Roles → Create role** → AWS service → Lambda.
2. Attach `AWSLambdaBasicExecutionRole` (write Lambda logs to CloudWatch).
3. Role name: `LambdaNotesExecutionRole`.
4. Add an **inline policy** `NotesTableAccess` scoped to the `Notes` table:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:GetItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem",
        "dynamodb:Scan",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:<ACCOUNT_ID>:table/Notes"
    }
  ]
}
```

Replace `<ACCOUNT_ID>` with your 12-digit account ID.

## Step 3 — GitHub Actions deploy role

Role name used in CI: `GitHubActionsCloudNoteDeploy`.

**Trust policy:** federated principal `token.actions.githubusercontent.com` (OIDC), scoped to the CloudNote repository (`thuydung0511/CloudNote` — will be updated soon).

**Permissions policy** (summary):

- `s3:PutObject`, `s3:DeleteObject`, `s3:ListBucket` on the app bucket.
- `lambda:UpdateFunctionCode`, `lambda:PublishVersion` on `notes-api`.
- `dynamodb:GetItem`, `dynamodb:PutItem` as needed for integration tests.
- `iam:PassRole` where a service role is required by deploy tooling.

## Step 4 — CodeDeploy / EC2 service roles (Part B)

| Role | Used for |
|------|----------|
| EC2 instance profile | Httpd web tier instances pulling content / registering with the target group |
| `AWSCodeDeployRole` (if used) | Deploying to the EC2 fleet behind the ALB |

## Verification

- `cloudnote-dev` can perform every Part A step without touching the root account.
- `LambdaNotesExecutionRole` has exactly two policies: `AWSLambdaBasicExecutionRole` + `NotesTableAccess`.
- GitHub Actions workflow assumes the deploy role without static access keys.