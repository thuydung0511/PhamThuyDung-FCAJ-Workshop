---
title: "Tổng quan workshop"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Mục tiêu

Xây dựng **CloudNote**, ứng dụng web ghi chú cá nhân trên AWS: người dùng đăng ký, đăng nhập, rồi thêm, sửa, xoá và nghe đọc ghi chú của mình. Mỗi người chỉ xem và thao tác được trên ghi chú của chính mình. Toàn bộ chạy trên dịch vụ serverless, trong phạm vi Free Tier.

## Kiến trúc

Trình duyệt tải trang web tĩnh (HTML/CSS/JS) từ **Amazon S3**. Người dùng đăng nhập bằng **Amazon Cognito** và nhận JWT. Mọi lời gọi tới **API Gateway** (HTTP API) đều kèm JWT; **JWT Authorizer** kiểm tra token trước khi chuyển request tới **AWS Lambda** `notes-api`. Lambda đọc/ghi bảng **Amazon DynamoDB** `Notes`, luôn lọc theo `userId` của người gọi, và gọi **Amazon Polly** để tạo giọng đọc cho tính năng "Đọc ghi chú". **Amazon CloudWatch** giám sát và cảnh báo lỗi, **AWS CloudTrail** ghi lại các lời gọi API trong tài khoản.

Phần B (VPC, EC2, ALB, Auto Scaling) là bài thực hành riêng về web tier có khả năng mở rộng ngang. Phần này đã được dọn dẹp sau khi kiểm thử và không thuộc kiến trúc CloudNote.

> Sơ đồ kiến trúc: sẽ cập nhật.

## Điều kiện

- Tài khoản AWS (Free account plan), Region `ap-southeast-1` (Singapore).
- Thao tác trên AWS Console; test API bằng PowerShell trên máy cá nhân.
- Không dùng tài khoản root cho công việc hằng ngày (xem [5.2](../5.2-iam/)).

## Tên tài nguyên

| Dịch vụ | Tài nguyên |
|---------|------------|
| IAM | User `cloudnote-dev`; group `cloudnote-network-group`; role `LambdaNotesExecutionRole` (inline policy `NotesTableAccess`, `PollySynthesizeSpeech`) |
| DynamoDB | Bảng `Notes` (partition key `noteId`) |
| Lambda | `notes-api` (Python 3.12) |
| API Gateway | HTTP API `notes-http-api` |
| S3 | Bucket web tĩnh `cloudnote-app-0205568-2026` |
| Cognito | 1 User Pool (đăng nhập bằng email), 1 App Client (SPA) |
| Polly | Engine Standard, giọng Joanna, định dạng MP3 |
| CloudWatch | Dashboard `CloudNote-Dashboard`, alarm `notes-api-error-alarm` |
| SNS | Topic `cloudnote-alerts` |
| CloudTrail | Trail `cloudnote-audit-trail` |
| Phần B (đã xoá) | `cloudnote-vpc`, `cloudnote-web-template`, `cloudnote-web-sg`, `cloudnote-key`, `cloudnote-tg`, `cloudnote-alb`, `cloudnote-asg` |

## Thứ tự thực hiện

1. [IAM](../5.2-iam/): user làm việc, role cho Lambda, group khi chạm giới hạn policy.
2. [Backend](../5.3-backend/): DynamoDB, Lambda, API Gateway, test API.
3. [Frontend](../5.4-frontend/): S3 static website, test end-to-end.
4. [Xác thực](../5.5-authentication/): Cognito, JWT Authorizer, cách ly dữ liệu theo người dùng.
5. ["Đọc ghi chú"](../5.6-polly-read-note/): Amazon Polly.
6. [Giám sát](../5.7-monitoring/): CloudWatch, SNS, CloudTrail.
7. [Phần B](../5.8-part-b-web-tier/): VPC, ALB, Auto Scaling.
8. [Dọn dẹp và chi phí](../5.9-cleanup-cost/).

Các giá trị thật (account ID, User Pool ID, App Client ID, Invoke URL, DNS name của ALB) được thay bằng placeholder như `<ACCOUNT_ID>`, `<USER_POOL_ID>`, `<APP_CLIENT_ID>`, `<API_URL>`, `<REGION>`, `<ALB_DNS_NAME>`.
