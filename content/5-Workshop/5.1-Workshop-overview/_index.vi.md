---
title: "Tổng quan workshop"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Mục đích

Workshop ghi lại các bước triển khai AWS cho đồ án capstone **CloudNote** — ứng dụng **ghi chú serverless** trên AWS Free Tier. Người dùng mở một trang web tĩnh (host trên **S3**), sau đó thêm, sửa, xem và xoá ghi chú qua giao diện web; toàn bộ dữ liệu được lưu và xử lý serverless (không cần quản lý máy chủ).

| Phần | Thành phần | Phần workshop của em |
|------|------------|----------------------|
| **A** | S3 frontend, API Gateway, Lambda, DynamoDB | IAM user, bảng DynamoDB `Notes`, Lambda `notes-api`, HTTP API + CORS, S3 static hosting, CloudWatch, CloudTrail |
| **B** | VPC, EC2, ALB, Auto Scaling | VPC tuỳ chỉnh, launch template, target group + ALB, ASG, kiểm thử cân bằng tải |

Phần A là deliverable lõi của capstone; Phần B là web tier bonus có khả năng mở rộng ngang.

## Điều kiện tiên quyết

- Tài khoản AWS (khuyến khích tài khoản mới để hưởng Free Tier 12 tháng) quyền admin tại `ap-southeast-1`
- Region đặt tại `ap-southeast-1`
- Chỉ cần trình duyệt — toàn bộ workshop thao tác trên **AWS Console** (AWS CLI không bắt buộc)
- Repository mã nguồn CloudNote: *sẽ cập nhật sau*

## Tên tài nguyên tham chiếu

| Tài nguyên | Tên |
|------------|-----|
| IAM user thực hành | `cloudnote-dev` |
| Bảng DynamoDB | `Notes` (partition key `noteId`) |
| Lambda function | `notes-api` (Python 3.12) |
| Execution role Lambda | `LambdaNotesExecutionRole` (+ inline `NotesTableAccess`) |
| API Gateway (HTTP API) | `notes-http-api` |
| S3 bucket | `cloudnote-app-0205568-2026` |
| CloudWatch dashboard / alarm | `CloudNote-Dashboard` / `notes-api-error-alarm` |
| CloudTrail trail | `cloudnote-audit-trail` |
| VPC | `cloudnote-vpc` |
| Launch template | `cloudnote-web-template` |
| Target group / ALB / ASG | `cloudnote-tg` / `cloudnote-alb` / `cloudnote-asg` |

## Thứ tự thực hiện

Làm **Phần A** trước (A0 → A9) để có ứng dụng full-stack serverless chạy thật, rồi làm **Phần B** (B1 → B5) cho web tier bonus. Ngay sau khi xong Phần B hãy thực hiện **dọn dẹp tài nguyên** theo tài liệu để tránh phí ALB.

## Checklist xác minh

- [ ] Bảng DynamoDB `Notes` ở trạng thái `Active` và nhận được dữ liệu
- [ ] Lambda `notes-api` trả về 201 khi test POST từ console
- [ ] Các route API Gateway (`GET/POST/PUT/DELETE /notes`) gọi được Lambda; CORS đã bật
- [ ] S3 website load được giao diện CloudNote; thêm/xoá ghi chú end-to-end
- [ ] CloudWatch dashboard hiển thị invocations/errors thật; alarm ở trạng thái OK
- [ ] CloudTrail trail đang `Logging`
- [ ] (Phần B) DNS của ALB luân phiên giữa 2 instance EC2 trong ASG