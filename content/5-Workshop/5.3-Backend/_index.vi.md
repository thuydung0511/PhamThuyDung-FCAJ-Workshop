---
title: "Backend: DynamoDB, Lambda, API Gateway"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Mục tiêu

Xây dựng backend CRUD cho CloudNote: bảng DynamoDB lưu ghi chú, một hàm Lambda xử lý GET/POST/PUT/DELETE và HTTP API trên API Gateway để trình duyệt gọi tới.

## Các bước

### 1. Bảng DynamoDB `Notes`

1. **DynamoDB → Tables → Create table**.
2. Tên bảng `Notes`, partition key `noteId` (String).
3. Chế độ capacity: **On-demand**.

### 2. Lambda `notes-api`

1. **Lambda → Create function → Author from scratch**.
2. Tên `notes-api`, runtime **Python 3.12**, handler `lambda_function.lambda_handler`.
3. Execution role: `LambdaNotesExecutionRole` ([5.2](../5.2-iam/)).
4. Viết code CRUD, **Deploy**, tạo Test event và chạy thử; kiểm tra item vừa tạo trong **DynamoDB → Explore table items**.

Code dưới đây là phiên bản cuối trong `lambda_function.py`. Phần `userId` được bổ sung ở bước xác thực ([5.5](../5.5-authentication/)), nhánh `/speak` ở [5.6](../5.6-polly-read-note/).

Phần khai báo và hàm dùng chung:

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

Hàm `lambda_handler` với 4 thao tác CRUD (lược bỏ nhánh `/speak`, xem 5.6):

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

1. **API Gateway → Create API → HTTP API → Build**, tên `notes-http-api`, integration Lambda `notes-api`.
2. Tạo 4 route, cùng trỏ tới `notes-api`:

| Method | Route |
|--------|-------|
| GET | `/notes` |
| POST | `/notes` |
| PUT | `/notes/{noteId}` |
| DELETE | `/notes/{noteId}` |

3. Dùng stage `$default` (Auto-deploy), lấy **Invoke URL**.
4. Bật **CORS**: Origin `*`, header `content-type`, method `GET, POST, PUT, DELETE, OPTIONS`. (Header `Authorization` được thêm ở [5.5](../5.5-authentication/).)

### 4. Test API bằng PowerShell

1. Mở `GET <API_URL>/notes` bằng trình duyệt.
2. Test đủ 4 method bằng `Invoke-RestMethod`, theo vòng GET → POST → GET → PUT → GET → DELETE → GET.

   **Ví dụ lệnh:**

```powershell
$api = "<API_URL>/notes"
Invoke-RestMethod -Uri $api -Method Get
Invoke-RestMethod -Uri $api -Method Post -ContentType "application/json" -Body '{"title":"Test","content":"Hello"}'
Invoke-RestMethod -Uri "$api/<noteId>" -Method Put -ContentType "application/json" -Body '{"title":"Test","content":"Updated"}'
Invoke-RestMethod -Uri "$api/<noteId>" -Method Delete
```

3. Test CORS bằng một request `OPTIONS` giả lập preflight của trình duyệt.

## Lỗi gặp phải và cách xử lý

| Lỗi | Cách xử lý |
|-----|------------|
| CloudShell báo "insufficient permissions" | Gắn `AWSCloudShellFullAccess` cho user |
| CloudShell báo tài khoản đang được xác minh (tới 2 ngày) | Giới hạn của tài khoản mới, không tự sửa được → test bằng PowerShell trên máy cá nhân (API là public endpoint) |
| `GET /notes` trả về sai thứ tự thời gian | `Scan()` không đảm bảo thứ tự → sắp xếp theo `createdAt` giảm dần trong Lambda, deploy lại, test bằng 3 ghi chú liên tiếp |

## Kết quả kiểm thử

- 4 route CRUD hoạt động; dữ liệu được ghi, sửa, xoá thật trong DynamoDB; không có lỗi 500/502/403.
- PUT giữ nguyên `createdAt`.
- Ghi chú mới nhất luôn nằm đầu danh sách.
- Preflight `OPTIONS` trả về 204 kèm đủ 3 header `Access-Control-Allow-Origin/Methods/Headers`.
