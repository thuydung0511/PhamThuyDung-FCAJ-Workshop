---
title: "Worklog Tuần 4"
date: 2026-09-12
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

* Tìm hiểu AWS Lambda và mô hình serverless.
* Tìm hiểu giám sát và kiểm toán với Amazon CloudWatch và AWS CloudTrail.
* Thiết kế kiến trúc CloudNote.
* Bắt đầu thực hành CloudNote: IAM user, bảng DynamoDB và execution role cho Lambda (A0–A2).

**Thời gian:** 12/09/2026 – 19/09/2026

### Các công việc đã thực hiện trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | ------------ | --------------- | -------------- |
| 7 | Tham gia buổi học tại văn phòng AWS Hà Nội | 12/09/2026 | 12/09/2026 | |
| 7 | Tham gia buổi học tại văn phòng AWS Hà Nội | 19/09/2026 | 19/09/2026 | |
| 7 | Bắt đầu thực hành CloudNote (A0–A2): IAM user `cloudnote-dev`, bảng DynamoDB `Notes`, IAM role `LambdaNotesExecutionRole` | 19/09/2026 | 19/09/2026 | |

Tự học trong tuần: Lambda, API Gateway, DynamoDB, CloudWatch và CloudTrail; thiết kế kiến trúc CloudNote.

### Kết quả đạt được tuần 4:

* Hiểu cách Lambda được kích hoạt, nhận quyền qua execution role và ghi log lên CloudWatch.
* Phân biệt CloudWatch (metric, log, alarm) và CloudTrail (ghi lại các lời gọi API để kiểm toán).
* Thiết kế kiến trúc CloudNote:
  * Frontend tĩnh lưu trên Amazon S3.
  * API CRUD trên API Gateway (HTTP API) gọi một hàm Lambda.
  * Dữ liệu ghi chú lưu trong bảng DynamoDB.
  * Giám sát bằng CloudWatch, kiểm toán bằng CloudTrail.
* Lập danh sách tài nguyên cần tạo và thứ tự triển khai.
* Bắt đầu thực hành từ 19/09/2026:
  * **A0 – IAM:** tạo IAM user `cloudnote-dev` với 7 managed policy (S3, DynamoDB, Lambda, API Gateway, IAM, CloudWatch, CloudTrail); từ đây làm việc bằng user này, không dùng root.
  * **A1 – DynamoDB:** bảng `Notes`, partition key `noteId` (String), chế độ On-demand.
  * **A2 – IAM Role:** `LambdaNotesExecutionRole` gồm `AWSLambdaBasicExecutionRole` và inline policy `NotesTableAccess`, chỉ cho phép Put/Get/Update/Delete/Scan/Query trên bảng `Notes`.
* Làm tiếp A3–A8 trong hai ngày 19–20/09, xem [tuần 5](../1.5-week5/).
