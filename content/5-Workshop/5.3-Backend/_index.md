---
title: "Backend: DynamoDB, Lambda, API Gateway"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Goal

Build the CRUD backend for CloudNote: a DynamoDB table for notes, a Lambda function handling GET/POST/PUT/DELETE, and an HTTP API on API Gateway for the browser to call.

## Steps

### 1. DynamoDB table `Notes`

1. **DynamoDB → Tables → Create table**.
2. Table name `Notes`, partition key `noteId` (String).
3. Capacity mode: **On-demand**.

### 2. Lambda `notes-api`

1. **Lambda → Create function → Author from scratch**.
2. Name `notes-api`, runtime **Python 3.12**, handler `lambda_function.lambda_handler`.
3. Execution role: `LambdaNotesExecutionRole` ([5.2](../5.2-iam/)).
4. Write the CRUD code, **Deploy**, create a Test event and run it; check the new item in **DynamoDB → Explore table items**.

The code below is the final version of `lambda_function.py`. The `userId` parts were added during the authentication step ([5.5](../5.5-authentication/)), and the `/speak` branch in [5.6](../5.6-polly-read-note/).

Declarations and shared helpers:

```python
import json
import base64
import os
import uuid
import boto3
from decimal import Decimal
from boto3.dynamodb.conditions import Key, Attr

dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table("Notes")
polly = boto3.client("polly")

HEADERS = {
    "Access-Control-Allow-Origin": "*",
    "Access-Control-Allow-Headers": "Content-Type,Authorization",
    "Access-Control-Allow-Methods": "GET,POST,PUT,DELETE,OPTIONS",
    "Content-Type": "application/json",
}

def decimal_default(obj):
    if isinstance(obj, Decimal):
        return int(obj)
    raise TypeError

def response(status, body):
    return {"statusCode": status, "headers": HEADERS, "body": json.dumps(body, default=decimal_default)}
```

`lambda_handler` with the 4 CRUD operations (the `/speak` branch is omitted, see 5.6):

```python
def lambda_handler(event, context):
    method = event.get("requestContext", {}).get("http", {}).get("method", "GET")
    path_params = event.get("pathParameters") or {}
    note_id = path_params.get("noteId")
    user_id = get_user_id(event)

    try:
        if method == "OPTIONS":
            return response(200, {})

        if user_id is None:
            return response(401, {"error": "unauthorized"})

        if method == "GET" and not note_id:
            items = table.scan(FilterExpression=Attr("userId").eq(user_id)).get("Items", [])
            items.sort(key=lambda x: x.get("createdAt", 0), reverse=True)
            return response(200, items)

        if method == "POST":
            body = json.loads(event.get("body") or "{}")
            title = body.get("title", "").strip()
            content = body.get("content", "").strip()
            if not title:
                return response(400, {"error": "title is required"})
            item = {
                "noteId": str(uuid.uuid4()),
                "userId": user_id,
                "title": title,
                "content": content,
                "createdAt": Decimal(str(__import__("time").time())),
            }
            table.put_item(Item=item)
            return response(201, item)

        if method == "PUT" and note_id:
            body = json.loads(event.get("body") or "{}")
            try:
                table.update_item(
                    Key={"noteId": note_id},
                    UpdateExpression="SET title = :t, content = :c",
                    ConditionExpression="userId = :uid",
                    ExpressionAttributeValues={
                        ":t": body.get("title", ""),
                        ":c": body.get("content", ""),
                        ":uid": user_id,
                    },
                )
            except dynamodb.meta.client.exceptions.ConditionalCheckFailedException:
                return response(403, {"error": "forbidden"})
            return response(200, {"message": "updated"})

        if method == "DELETE" and note_id:
            try:
                table.delete_item(
                    Key={"noteId": note_id},
                    ConditionExpression="userId = :uid",
                    ExpressionAttributeValues={":uid": user_id},
                )
            except dynamodb.meta.client.exceptions.ConditionalCheckFailedException:
                return response(403, {"error": "forbidden"})
            return response(200, {"message": "deleted"})

        return response(404, {"error": "route not found"})

    except Exception as e:
        return response(500, {"error": str(e)})
```

### 3. API Gateway `notes-http-api`

1. **API Gateway → Create API → HTTP API → Build**, name `notes-http-api`, Lambda integration `notes-api`.
2. Create 4 routes, all pointing to `notes-api`:

| Method | Route |
|--------|-------|
| GET | `/notes` |
| POST | `/notes` |
| PUT | `/notes/{noteId}` |
| DELETE | `/notes/{noteId}` |

3. Use the `$default` stage (Auto-deploy) and copy the **Invoke URL**.
4. Enable **CORS**: origin `*`, header `content-type`, methods `GET, POST, PUT, DELETE, OPTIONS`. (The `Authorization` header is added in [5.5](../5.5-authentication/).)

### 4. Test the API with PowerShell

1. Open `GET <API_URL>/notes` in the browser.
2. Test all 4 methods with `Invoke-RestMethod`, in the cycle GET → POST → GET → PUT → GET → DELETE → GET.

   **Example commands:**

```powershell
$api = "<API_URL>/notes"
Invoke-RestMethod -Uri $api -Method Get
Invoke-RestMethod -Uri $api -Method Post -ContentType "application/json" -Body '{"title":"Test","content":"Hello"}'
Invoke-RestMethod -Uri "$api/<noteId>" -Method Put -ContentType "application/json" -Body '{"title":"Test","content":"Updated"}'
Invoke-RestMethod -Uri "$api/<noteId>" -Method Delete
```

3. Test CORS with an `OPTIONS` request that simulates the browser preflight.

## Issues and fixes

| Issue | Fix |
|-------|-----|
| CloudShell reported "insufficient permissions" | Attached `AWSCloudShellFullAccess` to the user |
| CloudShell reported the account was being verified (up to 2 days) | A limitation of new accounts that cannot be fixed → tested with PowerShell on my own machine (the API is a public endpoint) |
| `GET /notes` returned notes in the wrong order | `Scan()` does not guarantee ordering → sorted by `createdAt` descending in Lambda, redeployed, tested with 3 consecutive notes |

## Test results

- All 4 CRUD routes work; data is really created, updated and deleted in DynamoDB; no 500/502/403 errors.
- PUT keeps the original `createdAt`.
- The newest note is always first in the list.
- The `OPTIONS` preflight returns 204 with all 3 headers `Access-Control-Allow-Origin/Methods/Headers`.
