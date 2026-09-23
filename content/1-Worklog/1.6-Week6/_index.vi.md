---
title: "Worklog Tuần 6"
date: 2026-09-05
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:

* Bắt đầu dự án capstone **CloudNote** — ứng dụng ghi chú serverless trên AWS Free Tier.
* Xây dựng phần lõi serverless: IAM user thực hành, bảng DynamoDB và backend Lambda CRUD.

**Thời gian:** 05/09/2026 – 11/09/2026

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | ------------ | --------------- | -------------- |
| 2 | - Tham dự kickoff dự án: tổng quan kiến trúc CloudNote, deliverable | 05/09/2026 | 05/09/2026 | Brief dự án FCAJ |
| 3 | - Tạo IAM user thực hành `cloudnote-dev` (không dùng root) | 06/09/2026 | 06/09/2026 | |
| 4 | - Tạo bảng DynamoDB `Notes` (partition key `noteId`, on-demand) | 07/09/2026 | 07/09/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Tạo execution role `LambdaNotesExecutionRole` với policy `NotesTableAccess` | 08/09/2026 | 08/09/2026 | |
| 6 | - Triển khai Lambda `notes-api` (Python 3.12) xử lý GET/POST/PUT/DELETE | 09/09/2026 | 09/09/2026 | |

### Kết quả đạt được tuần 6:

* Khởi động capstone **CloudNote** — ứng dụng ghi chú full-stack serverless: S3 frontend + API Gateway + Lambda + DynamoDB.
* Tạo IAM user thực hành `cloudnote-dev` tuân thủ nguyên tắc least-privilege và không dùng root.
* Cấp phát bảng DynamoDB `Notes` với on-demand capacity và partition key `noteId`.
* Xây dựng và deploy Lambda `notes-api` thực hiện CRUD trên DynamoDB.
* Xây dựng nền tảng serverless sẽ được exposed qua API Gateway ở Tuần 7.