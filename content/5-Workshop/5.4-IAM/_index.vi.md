---
title: "IAM"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Mục tiêu

Cấu hình IAM cho ứng dụng serverless **CloudNote**: user thực hành không dùng root, role `LambdaNotesExecutionRole` đọc/ghi bảng `Notes`, và role deploy GitHub Actions qua OIDC.

## Bước 1 — IAM user thực hành (không dùng root)

1. **IAM → Users → Create user**. User name: `cloudnote-dev`.
2. Chọn **Attach policies directly** và gắn các policy thực hành (phạm vi rộng để học, không dùng production):
   - `AmazonS3FullAccess`, `AmazonDynamoDBFullAccess`, `AWSLambda_FullAccess`
   - `AmazonAPIGatewayAdministrator`, `IAMFullAccess`, `CloudWatchFullAccess`, `AWSCloudTrail_FullAccess`
   - (Chỉ Phần B) `AmazonVPCFullAccess`, `AmazonEC2FullAccess`, `ElasticLoadBalancingFullAccess`
3. Bật **Console access** cho user này, đăng xuất root.
4. Dùng `cloudnote-dev` cho toàn bộ workshop.

## Bước 2 — Execution role Lambda

1. **IAM → Roles → Create role** → AWS service → Lambda.
2. Gắn `AWSLambdaBasicExecutionRole` (ghi log CloudWatch).
3. Role name: `LambdaNotesExecutionRole`.
4. Tạo **inline policy** `NotesTableAccess` giới hạn CRUD trên bảng `Notes`:

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

Thay `<ACCOUNT_ID>` bằng số tài khoản 12 chữ số của bạn.

## Bước 3 — GitHub Actions deploy role

Role dùng trong CI: `GitHubActionsCloudNoteDeploy`.

**Trust policy:** principal federated `token.actions.githubusercontent.com` (OIDC), giới hạn repository CloudNote (`thuydung0511/CloudNote` — sẽ cập nhật sau).

**Permissions policy** (tóm tắt):

- `s3:PutObject`, `s3:DeleteObject`, `s3:ListBucket` trên bucket ứng dụng.
- `lambda:UpdateFunctionCode`, `lambda:PublishVersion` trên `notes-api`.
- `dynamodb:GetItem`, `dynamodb:PutItem` nếu cần cho integration test.
- `iam:PassRole` khi tool deploy yêu cầu service role.

## Bước 4 — Service role EC2 (Phần B)

| Role | Dùng cho |
|------|----------|
| Instance profile EC2 | Instance web tier httpd lấy nội dung / đăng ký với target group |
| `AWSCodeDeployRole` (nếu dùng) | Deploy lên fleet EC2 phía sau ALB |

## Xác minh

- `cloudnote-dev` thực hiện được mọi bước Phần A mà không cần root.
- `LambdaNotesExecutionRole` có đúng 2 policy: `AWSLambdaBasicExecutionRole` + `NotesTableAccess`.
- Workflow GitHub Actions assume deploy role mà không cần access key tĩnh.