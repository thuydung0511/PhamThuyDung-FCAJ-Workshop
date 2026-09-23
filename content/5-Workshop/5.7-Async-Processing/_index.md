---
title: "Monitoring & auditing (CloudWatch, CloudTrail)"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Goal

Observe and audit the **CloudNote** workload: **CloudWatch** tracks Lambda and DynamoDB metrics and raises an alarm on Lambda errors; **CloudTrail** records every management API call in the account.

![CloudWatch / CloudTrail overview](/images/5-Workshop/image30.png)

## Step 1 — Create the CloudWatch dashboard

1. **CloudWatch → Dashboards → Create dashboard** → name `CloudNote-Dashboard`.
2. Add a **Line** widget:
   - **Metrics → Lambda → By Function Name** → tick `Invocations`, `Errors`, `Duration` of `notes-api`.
3. Add a second widget:
   - **Metrics → DynamoDB** → `ConsumedReadCapacityUnits`, `ConsumedWriteCapacityUnits` of table `Notes`.
4. **Save dashboard**.

![CloudWatch dashboard](/images/5-Workshop/image31.png)

## Step 2 — Create a Lambda error alarm

1. **CloudWatch → Alarms → All alarms → Create alarm**.
2. **Select metric → Lambda → By Function Name** → select `Errors` of `notes-api`.
3. Statistic **Sum**, Period **5 minutes**.
4. Condition: **Greater than** threshold `0`.
5. Notification: create SNS topic `cloudnote-alerts`, add your email (confirm the subscription).
6. **Alarm name:** `notes-api-error-alarm` → create.

## Step 3 — Verify the alarm state

- With no errors, the alarm state is **OK** (green).
- Trigger a deliberate error (e.g. a `GET /notes/{missing}` route) to see it briefly move to **IN ALARM** — this proves the monitoring chain works.

## Step 4 — Enable CloudTrail

1. **CloudTrail → Trails → Create trail**.
2. **Trail name:** `cloudnote-audit-trail`.
3. **Storage location:** create new S3 bucket, e.g. `cloudnote-trail-logs-0205568`.
4. Keep **Management events = Read/Write: All** → create trail.

> Management events of the **first trail** are free forever in CloudTrail — create only one trail.

## Step 5 — Verify the trail

- **Trail status = Logging**.
- After some API calls, open the trail's S3 bucket to confirm log files appear.

## Expected outcome

- Metrics for Lambda/DynamoDB visible on the dashboard
- Alarm fires on Lambda errors with an email notification
- Every management API call recorded by CloudTrail for audit

## Troubleshooting

| Issue | Action |
|-------|--------|
| No metrics on the dashboard | Ensure the function/table is in `ap-southeast-1`; widget shows the right region |
| Alarm stuck in INSUFFICIENT_DATA | Wait for the first 5-minute period; metric names must match `notes-api` |
| CloudTrail bucket empty | Status must be **Logging**; allow a few minutes after activity |