---
title: "S3 Hosting"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Mục tiêu

Host **frontend CloudNote** (HTML/CSS/JS) trên **S3 static website hosting**. Trình duyệt tải trang từ S3 và gọi API Gateway; CORS được xử lý ở API Gateway.

## Bước 1 — Tạo bucket

1. **S3 → Create bucket**.
2. **Bucket name:** `cloudnote-app-0205568-2026` (đặt tên duy nhất toàn cầu của riêng bạn).
3. **Region:** `ap-southeast-1`.
4. **Block Public Access:** **bỏ tick** "Block all public access" (đây là website công khai) và xác nhận cảnh báo.

![Tạo S3 bucket](/images/5-Workshop/image6.png)

5. Create bucket.

## Bước 2 — Bật static website hosting

1. Vào bucket → **Properties → Static website hosting → Edit → Enable**.
2. **Index document:** `index.html`.
3. **Error document:** `index.html` (để refresh theo kiểu SPA không lỗi).
4. Save và ghi lại **bucket website endpoint**.

![Static website hosting](/images/5-Workshop/image7.png)

## Bước 3 — Bucket policy (public read)

1. **Permissions → Bucket policy → Edit**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cloudnote-app-0205568-2026/*"
    }
  ]
}
```

2. Dùng đúng tên bucket bạn đã tạo; Save.

![Bucket policy (public read)](/images/5-Workshop/image8.png)

## Bước 4 — Chuẩn bị frontend

Tạo `index.html` trên máy với giao diện CloudNote (theo tài liệu capstone). Dòng quan trọng đặt URL API:

```html
const API_URL = "https://xxxxxx.execute-api.ap-southeast-1.amazonaws.com/notes";
```

Thay bằng **Invoke URL** API Gateway của bạn ở [5.6](5.6-CodeDeploy/). Thiếu bước này, trình duyệt không gọi được Lambda backend.

## Bước 5 — Upload và kiểm tra

1. **S3 → bucket → Objects → Upload → Add files** → `index.html` → Upload.
2. Mở **bucket website endpoint** — giao diện CloudNote tải lên và hiển thị danh sách ghi chú hiện có (đã tạo ở [5.6](5.6-CodeDeploy/)).
3. Thêm một ghi chú và xoá một ghi chú để xác nhận toàn bộ luồng hoạt động.

## Kết quả mong đợi

- Website công khai trên S3 phục vụ frontend CloudNote
- Frontend gọi được API Gateway + Lambda không lỗi CORS
- Full stack hoạt động: browser → S3 → API Gateway → Lambda → DynamoDB

## Lưu ý về HTTPS

Endpoint S3 website là **HTTP**. Muốn HTTPS hoặc domain riêng, đặt **CloudFront** phía trước bucket.