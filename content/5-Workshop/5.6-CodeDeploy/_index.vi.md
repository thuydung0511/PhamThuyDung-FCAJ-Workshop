---
title: "Backend DynamoDB, Lambda & API Gateway"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Mục tiêu

Xây dựng **lõi backend serverless** của CloudNote (Phần A): bảng **DynamoDB**, function **Lambda** CRUD và **API Gateway HTTP API** expose ra ngoài. Đây chính là "động cơ" thêm/sửa/xem/xoá ghi chú của ứng dụng.

## Bước 1 — Tạo bảng DynamoDB

1. **DynamoDB → Tables → Create table**.
2. **Table name:** `Notes`.
3. **Partition key:** `noteId` — kiểu **String**.
4. **Customize settings → Read/write capacity:** **On-demand** (trả tiền theo request, không chi phí cố định).
5. **Create table**, đợi status **Active** (~30–60 giây).

## Bước 2 — Execution role Lambda (IAM)

1. **IAM → Roles → Create role → AWS service → Lambda**.
2. Gán `AWSLambdaBasicExecutionRole` (ghi log CloudWatch).
3. **Role name:** `LambdaNotesExecutionRole`.
4. Tạo **inline policy** `NotesTableAccess` (xem JSON ở [5.4 IAM](5.4-IAM/)) cho phép CRUD DynamoDB trên ARN bảng `Notes`.

## Bước 3 — Tạo Lambda function

1. **Lambda → Create function → Author from scratch**.
2. **Function name:** `notes-api` · **Runtime:** `Python 3.12` · **Architecture:** `x86_64`.
3. **Execution role:** dùng role có sẵn `LambdaNotesExecutionRole`.
4. Thay code mặc định bằng handler CRUD (xem tài liệu capstone) — một Lambda xử lý **GET** (danh sách), **POST** (tạo), **PUT** (sửa), **DELETE** theo HTTP method.
5. **Deploy**.

## Bước 4 — Test Lambda từ console

1. Tab **Test → Create new test event** → name `TestCreate`:

```json
{
  "requestContext": { "http": { "method": "POST" } },
  "body": "{\"title\":\"Ghi chú đầu tiên\",\"content\":\"Test Lambda\"}"
}
```

2. **Save → Test** → kỳ vọng `statusCode: 201`.
3. Xác nhận item xuất hiện trong **DynamoDB → Explore table items**.

## Bước 5 — Tạo API Gateway HTTP API

1. **API Gateway → Create API → HTTP API → Build**.
2. **API name:** `notes-http-api`.
3. **Integration:** thêm Lambda `notes-api`.
4. **Routes** (đều trỏ tới `notes-api`):

| Method | Path |
|--------|------|
| GET | `/notes` |
| POST | `/notes` |
| PUT | `/notes/{noteId}` |
| DELETE | `/notes/{noteId}` |

5. Dùng stage mặc định `$default` (tự deploy). Copy **Invoke URL**.
6. **Bật CORS** (bắt buộc cho frontend S3): `Access-Control-Allow-Origin: *`, methods `GET, POST, PUT, DELETE, OPTIONS`, headers `Content-Type`.

## Bước 6 — Xác minh API

1. Browser/curl: `GET <Invoke-URL>/notes` → danh sách JSON.
2. `curl -X POST <Invoke-URL>/notes -H "Content-Type: application/json" -d '{"title":"Note qua CloudShell","content":"POST test"}'` → JSON 201.

## Kết quả mong đợi

- Bảng `Notes` lưu và trả dữ liệu
- `notes-api` xử lý đủ 4 thao tác CRUD
- HTTP API expose Lambda kèm CORS cho frontend S3

## Xử lý sự cố

| Vấn đề | Cách khắc phục |
|--------|----------------|
| Lambda trả 500 | Xem CloudWatch Logs (`/aws/lambda/notes-api`) tìm traceback; kiểm tra ARN trong `NotesTableAccess` |
| API trả 403 | Role Lambda thiếu quyền DynamoDB; sửa inline policy ở 5.4 |
| Lỗi CORS trên trình duyệt | Redeploy stage `$default` sau khi bật CORS |