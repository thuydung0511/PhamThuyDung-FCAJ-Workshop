---
title: "IAM: user, role and group"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Goal

- Use a dedicated IAM user for daily work instead of the root account.
- Give Lambda only the permissions it needs on the `Notes` table (least privilege).
- Grant extra permissions through an IAM group once the user reaches the limit of 10 directly attached managed policies.

## Steps

### 1. IAM user `cloudnote-dev`

1. Sign in to the Console as root and switch the Region to `ap-southeast-1` (Singapore).
2. **IAM → Users → Create user**, name `cloudnote-dev`, enable Console access and set a password.
3. **Attach policies directly**, attach 7 managed policies for practice:
   - `AmazonS3FullAccess`, `AmazonDynamoDBFullAccess`, `AWSLambda_FullAccess`
   - `AmazonAPIGatewayAdministrator`, `IAMFullAccess`
   - `CloudWatchFullAccess`, `AWSCloudTrail_FullAccess`
4. Sign out of root; from here on, work as `cloudnote-dev`.

Later, the user also received `AWSCloudShellFullAccess` ([5.3](../5.3-backend/)) and `AmazonCognitoPowerUser` ([5.5](../5.5-authentication/)) after permission errors, and `AmazonBedrockFullAccess` when trying Bedrock ([5.6](../5.6-polly-read-note/)).

### 2. Role `LambdaNotesExecutionRole`

1. **IAM → Roles → Create role → AWS service → Lambda**.
2. Attach `AWSLambdaBasicExecutionRole` so Lambda can write logs to CloudWatch.
3. Name it `LambdaNotesExecutionRole`.
4. Add inline policy `NotesTableAccess`, allowing only 6 actions on the `Notes` table:

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
      "Resource": "arn:aws:dynamodb:<REGION>:<ACCOUNT_ID>:table/Notes"
    }
  ]
}
```

Later, the role received inline policy `PollySynthesizeSpeech` (only `polly:SynthesizeSpeech`) for the "Read note" feature ([5.6](../5.6-polly-read-note/)).

### 3. IAM group `cloudnote-network-group`

Part B needed VPC, EC2 and ELB permissions, but the user was close to the limit of 10 directly attached managed policies.

1. **IAM → User groups → Create group**, name `cloudnote-network-group`.
2. Attach `AmazonVPCFullAccess`, `AmazonEC2FullAccess`, `ElasticLoadBalancingFullAccess` to the group.
3. Add `cloudnote-dev` to the group.

Later, the Console permissions for Polly and AWS Support were also added to this group (inline policy `ConsoleTestPolly`, managed policy `AWSSupportAccess`), see [5.6](../5.6-polly-read-note/).

## Issues and fixes

| Issue | Fix |
|-------|-----|
| CloudShell reported "insufficient permissions" | Attached `AWSCloudShellFullAccess` to the user |
| Could not attach more policies: limit of 10 directly attached managed policies | Created group `cloudnote-network-group`, attached the policies to the group and added the user |
| `AccessDeniedException` when opening Cognito | Attached `AmazonCognitoPowerUser` |
| Missing `polly:DescribeVoices` and `support:DescribeSupportLevel` in the Console | Added inline policy `ConsoleTestPolly` and `AWSSupportAccess` to the group (without using root) |
| The IAM user could not view Billing (blocked by default) | Signed in as root once to create the Zero-Spend Budget, then signed out ([5.9](../5.9-cleanup-cost/)) |

## Results

- Every workshop step was done as `cloudnote-dev`; root was used only once to create the budget.
- `LambdaNotesExecutionRole` can only write logs and perform 6 actions on the `Notes` table (plus `polly:SynthesizeSpeech` later).
- Part B permissions and the extra Console permissions are managed centrally through the group.
