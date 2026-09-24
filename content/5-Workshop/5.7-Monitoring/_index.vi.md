---
title: "Giám sát và kiểm toán: CloudWatch, CloudTrail"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Mục tiêu

- Theo dõi số liệu của Lambda và DynamoDB trên một dashboard.
- Nhận email cảnh báo khi Lambda `notes-api` gặp lỗi.
- Ghi lại các lời gọi API trong tài khoản để kiểm toán.

## Các bước

### 1. CloudWatch Dashboard

1. **CloudWatch → Dashboards → Create dashboard**, tên `CloudNote-Dashboard`.
2. Widget 1 — Lambda `notes-api`: `Duration`, `Errors`, `Invocations`.
3. Widget 2 — DynamoDB bảng `Notes`: `ConsumedReadCapacityUnits`, `ConsumedWriteCapacityUnits`.
4. Lưu dashboard.

### 2. Alarm và SNS

1. **CloudWatch → Alarms → Create alarm**, metric `Errors` của `notes-api`.
2. Statistic **Sum**, period **5 phút**, điều kiện **> 0**.
3. Tạo SNS topic `cloudnote-alerts`, đăng ký email và xác nhận subscription qua email.
4. Tên alarm `notes-api-error-alarm`.
5. **Missing data treatment:** "Treat missing data as good".

### 3. CloudTrail

1. **CloudTrail → Trails → Create trail**, tên `cloudnote-audit-trail`, áp dụng cho mọi Region (multi-region).
2. Ghi **Management events** cả Read và Write; log được lưu vào một bucket S3 do trail tạo.
3. Kiểm tra trạng thái trail là **Logging**.

## Lỗi gặp phải và cách xử lý

| Lỗi | Cách xử lý |
|-----|------------|
| Alarm mới tạo ở trạng thái "Insufficient data" | Không có lượt gọi liên tục nên thiếu datapoint → đặt Missing data treatment = "Treat missing data as good"; alarm chuyển sang OK sau khoảng 11 giây |

## Kết quả kiểm thử

- Dashboard hiển thị số liệu thật của Lambda và DynamoDB.
- Alarm ở trạng thái **OK**, Actions enabled; email đăng ký SNS đã được xác nhận.
- CloudTrail: trạng thái **Logging**, Management events = All.
