---
title: "DynamoDB, Lambda & API Gateway backend"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Goal

Build the **serverless backend core** of CloudNote (Part A): a **DynamoDB** table, a **Lambda** CRUD function and an **API Gateway HTTP API** that exposes it. This is the create/read/update/delete engine behind the notes app.

## Step 1 — Create the DynamoDB table

1. **DynamoDB → Tables → Create table**.
2. **Table name:** `Notes`.
3. **Partition key:** `noteId` — type **String**.
4. **Customize settings → Read/write capacity:** **On-demand** (pay per request, no fixed cost).
5. **Create table** and wait for status **Active** (~30–60 s).

## Step 2 — Lambda execution role (IAM)

1. **IAM → Roles → Create role → AWS service → Lambda**.
2. Attach `AWSLambdaBasicExecutionRole` (write logs to CloudWatch).
3. **Role name:** `LambdaNotesExecutionRole`.
4. Add an **inline policy** `NotesTableAccess` (see [5.4 IAM](5.4-IAM/) for the JSON) allowing DynamoDB CRUD on the `Notes` table ARN.

## Step 3 — Create the Lambda function

1. **Lambda → Create function → Author from scratch**.
2. **Function name:** `notes-api` · **Runtime:** `Python 3.12` · **Architecture:** `x86_64`.
3. **Execution role:** use existing `LambdaNotesExecutionRole`.
4. Replace the default code with the CRUD handler (see the capstone workshop) — one Lambda handles **GET** (list), **POST** (create), **PUT** (update), **DELETE** via the HTTP method.
5. **Deploy**.

## Step 4 — Test the Lambda from the console

1. Tab **Test → Create new test event** → name `TestCreate`:

```json
{
  "requestContext": { "http": { "method": "POST" } },
  "body": "{\"title\":\"Đầu tiên\",\"content\":\"Test Lambda\"}"
}
```

2. **Save → Test** → expect `statusCode: 201`.
3. Verify the item appears in **DynamoDB → Explore table items**.

## Step 5 — Create the API Gateway HTTP API

1. **API Gateway → Create API → HTTP API → Build**.
2. **API name:** `notes-http-api`.
3. **Integration:** add Lambda `notes-api`.
4. **Routes** (all → `notes-api`):

| Method | Path |
|--------|------|
| GET | `/notes` |
| POST | `/notes` |
| PUT | `/notes/{noteId}` |
| DELETE | `/notes/{noteId}` |

5. Use the default `$default` stage (auto-deploy). Copy the **Invoke URL**.
6. **Enable CORS** (required for the S3 frontend): `Access-Control-Allow-Origin: *`, methods `GET, POST, PUT, DELETE, OPTIONS`, headers `Content-Type`.

## Step 6 — Verify the API

1. Browser/curl: `GET <Invoke-URL>/notes` → JSON list.
2. `curl -X POST <Invoke-URL>/notes -H "Content-Type: application/json" -d '{"title":"Note về CLoudShell","content":"POST test"}'` → 201 JSON.

## Expected outcome

- `Notes` table stores and returns notes
- `notes-api` handles all four CRUD operations
- HTTP API exposes the Lambda with CORS enabled for the S3 frontend

## Troubleshooting

| Issue | Action |
|-------|--------|
| Lambda returns 500 | Check CloudWatch Logs (`/aws/lambda/notes-api`) for traceback; verify `NotesTableAccess` ARN |
| API returns 403 | Lambda role missing DynamoDB permission; fix inline policy in 5.4 |
| CORS error in browser | Redeploy the `$default` stage after enabling CORS |