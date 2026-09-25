---
title: "Monitoring and auditing: CloudWatch, CloudTrail"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Goal

- Track Lambda and DynamoDB metrics on one dashboard.
- Receive an email alert when the `notes-api` Lambda has errors.
- Record API calls in the account for auditing.

## Steps

### 1. CloudWatch Dashboard

1. **CloudWatch → Dashboards → Create dashboard**, name `CloudNote-Dashboard`.
2. Widget 1 — Lambda `notes-api`: `Duration`, `Errors`, `Invocations`.
3. Widget 2 — DynamoDB table `Notes`: `ConsumedReadCapacityUnits`, `ConsumedWriteCapacityUnits`.
4. Save the dashboard.

![CloudNote-Dashboard over 1 week](/images/5-Workshop/5.7-dashboard.png)

*Figure: CloudNote-Dashboard over 1 week: Lambda Errors = 0 (times in UTC).*

### 2. Alarm and SNS

1. **CloudWatch → Alarms → Create alarm**, metric `Errors` of `notes-api`.
2. Statistic **Sum**, period **5 minutes**, condition **> 0**.
3. Create SNS topic `cloudnote-alerts`, subscribe an email address and confirm the subscription from the email.
4. Alarm name `notes-api-error-alarm`.
5. **Missing data treatment:** "Treat missing data as good".

![Alarm notes-api-error-alarm](/images/5-Workshop/5.7-alarm.png)

*Figure: Alarm notes-api-error-alarm: state OK, Actions enabled, condition Errors > 0 within 5 minutes.*

### 3. CloudTrail

1. **CloudTrail → Trails → Create trail**, name `cloudnote-audit-trail`, applied to all Regions (multi-region).
2. Record **Management events**, both Read and Write; logs are stored in an S3 bucket created by the trail.
3. Check that the trail status is **Logging**.

![Trail cloudnote-audit-trail](/images/5-Workshop/5.7-cloudtrail.png)

*Figure: Trail cloudnote-audit-trail: status Logging, multi-region.*

## Issues and fixes

| Issue | Fix |
|-------|-----|
| The new alarm stayed in "Insufficient data" | There were no continuous invocations, so datapoints were missing → set Missing data treatment = "Treat missing data as good"; the alarm moved to OK after about 11 seconds |

## Test results

- The dashboard shows real Lambda and DynamoDB data.
- The alarm is **OK** with Actions enabled; the SNS email subscription is confirmed.
- CloudTrail: status **Logging**, Management events = All.
