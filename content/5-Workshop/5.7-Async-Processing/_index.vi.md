---
title: "Giám sát & kiểm toán (CloudWatch, CloudTrail)"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Mục tiêu

Quan sát và kiểm toán workload **CloudNote**: **CloudWatch** theo dõi metric Lambda/DynamoDB và cảnh báo khi Lambda gặp lỗi; **CloudTrail** ghi lại mọi lời gọi API quản trị trong tài khoản.

![Tổng quan CloudWatch / CloudTrail](/images/5-Workshop/image30.png)

## Bước 1 — Tạo CloudWatch dashboard

1. **CloudWatch → Dashboards → Create dashboard** → name `CloudNote-Dashboard`.
2. Thêm widget **Line**:
   - **Metrics → Lambda → By Function Name** → tick `Invocations`, `Errors`, `Duration` của `notes-api`.
3. Thêm widget thứ hai:
   - **Metrics → DynamoDB** → `ConsumedReadCapacityUnits`, `ConsumedWriteCapacityUnits` của bảng `Notes`.
4. **Save dashboard**.

![CloudWatch dashboard](/images/5-Workshop/image31.png)

## Bước 2 — Tạo alarm lỗi Lambda

1. **CloudWatch → Alarms → All alarms → Create alarm**.
2. **Select metric → Lambda → By Function Name** → chọn `Errors` của `notes-api`.
3. Statistic **Sum**, Period **5 minutes**.
4. Condition: **Greater than** threshold `0`.
5. Notification: tạo SNS topic `cloudnote-alerts`, nhập email của bạn (xác nhận subscription).
6. **Alarm name:** `notes-api-error-alarm` → create.

## Bước 3 — Xác minh trạng thái alarm

- Không có lỗi → alarm ở trạng thái **OK** (xanh).
- Cố tình gây một lỗi (ví dụ gọi `GET /notes/{missing}`) để thấy alarm chuyển sang **IN ALARM** — chứng minh chuỗi giám sát hoạt động.

## Bước 4 — Bật CloudTrail

1. **CloudTrail → Trails → Create trail**.
2. **Trail name:** `cloudnote-audit-trail`.
3. **Storage location:** tạo S3 bucket mới, ví dụ `cloudnote-trail-logs-0205568`.
4. Giữ **Management events = Read/Write: All** → create trail.

> Management events của **trail đầu tiên** miễn phí vĩnh viễn trong CloudTrail — chỉ nên tạo **một** trail duy nhất.

## Bước 5 — Xác minh trail

- **Trail status = Logging**.
- Sau vài lời gọi API, mở bucket S3 của trail để xác nhận log file xuất hiện.

## Kết quả mong đợi

- Metric Lambda/DynamoDB hiển thị trên dashboard
- Alarm kích hoạt khi Lambda lỗi và gửi email thông báo
- Mọi lời gọi API quản trị được CloudTrail ghi lại phục vụ kiểm toán

## Xử lý sự cố

| Vấn đề | Cách khắc phục |
|--------|----------------|
| Dashboard không có metric | Đảm bảo function/bảng ở `ap-southeast-1`; widget đúng region |
| Alarm kẹt INSUFFICIENT_DATA | Đợi hết chu kỳ 5 phút đầu; tên metric đúng `notes-api` |
| Bucket CloudTrail trống | Status phải là **Logging**; đợi vài phút sau khi có hoạt động |