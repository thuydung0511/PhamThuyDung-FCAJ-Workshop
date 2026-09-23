---
title: "S3 Hosting"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Goal

Host the **CloudNote frontend** (HTML/CSS/JS) from **S3 static website hosting**. The browser loads the page from S3 and calls the API Gateway endpoint; CORS is handled by API Gateway.

## Step 1 — Create the bucket

1. **S3 → Create bucket**.
2. **Bucket name:** `cloudnote-app-0205568-2026` (replace with your own globally unique name).
3. **Region:** `ap-southeast-1`.
4. **Block Public Access:** untick **Block all public access** (this is a public website) and confirm the warning.

![Create the S3 bucket](/images/5-Workshop/image6.png)

5. Create bucket.

## Step 2 — Enable static website hosting

1. Open the bucket → **Properties → Static website hosting → Edit → Enable**.
2. **Index document:** `index.html`.
3. **Error document:** `index.html` (SPA-style refresh).
4. Save and note the **bucket website endpoint**.

![Static website hosting](/images/5-Workshop/image7.png)

## Step 3 — Bucket policy (public read)

1. **Permissions → Bucket policy → Edit**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cloudnote-app-0205568-2026/*"
    }
  ]
}
```

2. Use the exact bucket name you created; save.

![Bucket policy (public read)](/images/5-Workshop/image8.png)

## Step 4 — Prepare the frontend

Create `index.html` locally with the CloudNote UI (see the capstone workshop). The key line sets the API URL:

```html
const API_URL = "https://xxxxxx.execute-api.ap-southeast-1.amazonaws.com/notes";
```

Replace it with your API Gateway **Invoke URL** from [5.6](5.6-CodeDeploy/). Without this, the browser cannot reach the Lambda backend.

## Step 5 — Upload and verify

1. **S3 → bucket → Objects → Upload → Add files** → `index.html` → Upload.
2. Open the **bucket website endpoint** — the CloudNote UI loads and lists the existing notes (created in [5.6](5.6-CodeDeploy/)).
3. Add a note and delete a note to confirm the whole flow works.

## Expected outcome

- Public S3 website serves the CloudNote frontend
- Frontend reaches API Gateway + Lambda without CORS errors
- Full stack works: browser → S3 → API Gateway → Lambda → DynamoDB

## Note on HTTPS

S3 website endpoints are **HTTP**. For HTTPS or a custom domain, place **CloudFront** in front of the bucket.