---
title: "IAM: user, role và group"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Mục tiêu

- Có một IAM user riêng để làm việc hằng ngày, không dùng tài khoản root.
- Cấp cho Lambda đúng quyền cần thiết trên bảng `Notes` (least privilege).
- Cấp thêm quyền qua IAM group khi user chạm giới hạn 10 managed policy gắn trực tiếp.

## Các bước

### 1. IAM user `cloudnote-dev`

1. Đăng nhập Console bằng root, đổi Region sang `ap-southeast-1` (Singapore).
2. **IAM → Users → Create user**, tên `cloudnote-dev`, bật quyền truy cập Console và đặt mật khẩu.
3. **Attach policies directly**, gắn 7 managed policy phục vụ thực hành:
   - `AmazonS3FullAccess`, `AmazonDynamoDBFullAccess`, `AWSLambda_FullAccess`
   - `AmazonAPIGatewayAdministrator`, `IAMFullAccess`
   - `CloudWatchFullAccess`, `AWSCloudTrail_FullAccess`
4. Đăng xuất root; từ đây làm việc bằng `cloudnote-dev`.

Trong quá trình làm, user được gắn thêm `AWSCloudShellFullAccess` ([5.3](../5.3-backend/)) và `AmazonCognitoPowerUser` ([5.5](../5.5-authentication/)) khi gặp lỗi thiếu quyền, cùng `AmazonBedrockFullAccess` khi thử Bedrock ([5.6](../5.6-polly-read-note/)).

### 2. Role `LambdaNotesExecutionRole`

1. **IAM → Roles → Create role → AWS service → Lambda**.
2. Gắn `AWSLambdaBasicExecutionRole` để Lambda ghi log lên CloudWatch.
3. Đặt tên `LambdaNotesExecutionRole`.
4. Thêm inline policy `NotesTableAccess`, chỉ cho phép 6 thao tác trên đúng bảng `Notes`:

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

Sau này role được thêm inline policy `PollySynthesizeSpeech` (chỉ gồm `polly:SynthesizeSpeech`) cho tính năng "Đọc ghi chú" ([5.6](../5.6-polly-read-note/)).

![Role LambdaNotesExecutionRole với 3 policy](/images/5-Workshop/5.2-iam-role.png)

*Hình: Role LambdaNotesExecutionRole với 3 policy: AWSLambdaBasicExecutionRole, NotesTableAccess và PollySynthesizeSpeech (least privilege).*

### 3. IAM group `cloudnote-network-group`

Phần B cần thêm quyền VPC, EC2 và ELB, nhưng user đã gần chạm giới hạn 10 managed policy gắn trực tiếp.

1. **IAM → User groups → Create group**, tên `cloudnote-network-group`.
2. Gắn `AmazonVPCFullAccess`, `AmazonEC2FullAccess`, `ElasticLoadBalancingFullAccess` vào group.
3. Thêm `cloudnote-dev` vào group.

Về sau, các quyền dùng trên Console cho Polly và AWS Support cũng được thêm vào group này (inline policy `ConsoleTestPolly`, managed policy `AWSSupportAccess`), xem [5.6](../5.6-polly-read-note/).

## Lỗi gặp phải và cách xử lý

| Lỗi | Cách xử lý |
|-----|------------|
| CloudShell báo "insufficient permissions" | Gắn thêm `AWSCloudShellFullAccess` cho user |
| Không gắn thêm được policy vì vượt giới hạn 10 managed policy gắn trực tiếp | Tạo group `cloudnote-network-group`, gắn policy vào group, thêm user vào group |
| `AccessDeniedException` khi mở Cognito | Gắn `AmazonCognitoPowerUser` |
| Thiếu quyền `polly:DescribeVoices`, `support:DescribeSupportLevel` trên Console | Thêm inline policy `ConsoleTestPolly` và `AWSSupportAccess` vào group (không dùng root) |
| IAM user không xem được Billing (mặc định AWS chặn) | Đăng nhập root một lần để tạo Zero-Spend Budget rồi đăng xuất ([5.9](../5.9-cleanup-cost/)) |

## Kết quả

- Mọi bước trong workshop đều làm bằng `cloudnote-dev`; root chỉ dùng một lần để tạo budget.
- `LambdaNotesExecutionRole` chỉ có quyền ghi log và 6 thao tác trên bảng `Notes` (sau này thêm `polly:SynthesizeSpeech`).
- Quyền cho Phần B và các quyền Console bổ sung được quản lý tập trung qua group.
