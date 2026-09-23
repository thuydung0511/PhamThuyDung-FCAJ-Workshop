---
title: "Worklog Tuần 7"
date: 2026-09-12
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

* Hoàn thiện ứng dụng CloudNote: expose API qua API Gateway, host frontend trên S3.
* Bổ sung giám sát, kiểm toán và thực hiện demo end-to-end cuối cùng.

**Thời gian:** 12/09/2026 – 14/09/2026

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | ------------ | --------------- | -------------- |
| 2 | - Tạo API `notes-http-api` (HTTP API) và cấu hình CORS | 12/09/2026 | 12/09/2026 | Workshop CloudNote |
| 3 | - Host frontend ứng dụng trên S3 static website; kiểm chứng luồng đầy đủ (tạo, xem, xóa ghi chú) | 13/09/2026 | 13/09/2026 | |
| 4 | - Thêm CloudWatch dashboard/alarm và CloudTrail trail; tổng kết kết quả 7 tuần thực tập | 14/09/2026 | 14/09/2026 | |

### Kết quả đạt được tuần 7:

* Expose Lambda `notes-api` qua **API Gateway HTTP API** và bật CORS cho frontend trên trình duyệt.
* Host frontend CloudNote trên S3 static website hosting và kiểm chứng luồng end-to-end đầy đủ.
* Thiết lập CloudWatch dashboard và alarm cùng CloudTrail trail để quan sát và kiểm toán workload.
* Hoàn thành demo end-to-end ứng dụng ghi chú serverless trên AWS Free Tier.
* Tổng kết hành trình 7 tuần thực tập gồm AWS cơ bản, thực hành lab, lưu trữ, giám sát, thao tác CLI và capstone CloudNote.