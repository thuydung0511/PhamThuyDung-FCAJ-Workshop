---
title: "App demo"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Live app

**Open the deployed demo:** the **S3 website endpoint** of bucket `cloudnote-app-0205568-2026` (from [5.3](5.3-S3-Hosting/)).

The browser loads the static frontend from **S3**; the frontend calls the **API Gateway HTTP API** (`/notes`), which invokes the **notes-api Lambda**, which reads/writes the **DynamoDB `Notes` table**. No servers to manage.

## Core flow

1. **Open the website** — the CloudNote UI lists existing notes (loaded via `GET /notes`).
2. **Create a note** — fill title + content, click **Thêm ghi chú**; a `POST /notes` persists a new item to DynamoDB and it appears in the list immediately.
3. **Delete a note** — click **Xoá**; a `DELETE /notes/{noteId}` removes it from the list and from DynamoDB.
4. **Refresh the page** — the data is still there, proving persistence happens server-side in DynamoDB, not in the browser.

This is the **complete capstone deliverable**: a full-stack serverless app running on AWS Free Tier.

## Second demo (Part B — optional)

Open the **ALB DNS name** of `cloudnote-alb` and refresh several times — the displayed **Instance ID alternates** between the two EC2 instances behind the Auto Scaling Group, demonstrating horizontal scaling and load balancing.

## Demo material for the report

- Screenshot of the architecture diagram.
- Screenshot of the CloudNote UI with 3–5 sample notes.
- Short video (OBS) showing: open web → add note → delete note → refresh (data persists).
- Screenshot of the CloudWatch dashboard with real Invocations/Errors/Duration.
- (Part B) Video refreshing the ALB URL, Instance ID alternates.
- Screenshot of the `Notes` DynamoDB table with real items.

## CORS troubleshooting

If the browser console shows a `CORS policy` error:
1. Re-open API Gateway **CORS** settings and confirm `Access-Control-Allow-Origin: *`.
2. Redeploy the `$default` stage manually and retry.