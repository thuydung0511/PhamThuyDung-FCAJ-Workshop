---
title: "Demo ứng dụng"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## App trực tiếp

**Mở demo đã deploy:** **S3 website endpoint** của bucket `cloudnote-app-0205568-2026` (từ [5.3](5.3-S3-Hosting/)).

Trình duyệt tải frontend tĩnh từ **S3**; frontend gọi **API Gateway HTTP API** (`/notes`), API Gateway gọi **Lambda notes-api**, Lambda đọc/ghi bảng **DynamoDB `Notes`**. Không cần quản lý máy chủ nào.

## Luồng chính

1. **Mở website** — giao diện CloudNote hiển thị danh sách ghi chú (load qua `GET /notes`).
2. **Thêm ghi chú** — nhập tiêu đề + nội dung, bấm **Thêm ghi chú**; `POST /notes` lưu item mới vào DynamoDB và item xuất hiện ngay trong danh sách.
3. **Xoá ghi chú** — bấm **Xoá**; `DELETE /notes/{noteId}` xoá khỏi danh sách và khỏi DynamoDB.
4. **Refresh trang** — dữ liệu vẫn còn, chứng minh dữ liệu được lưu server-side trong DynamoDB, không phải trong trình duyệt.

Đây là **deliverable capstone hoàn chỉnh**: một ứng dụng serverless full-stack chạy thật trên AWS Free Tier.

## Demo thứ hai (Phần B — tuỳ chọn)

Mở **DNS name** của `cloudnote-alb` và refresh nhiều lần — **Instance ID hiển thị đổi luân phiên** giữa 2 instance EC2 phía sau Auto Scaling Group, chứng minh khả năng mở rộng ngang và cân bằng tải.

## Tư liệu demo cho báo cáo

- Ảnh chụp sơ đồ kiến trúc.
- Ảnh chụp giao diện CloudNote với 3–5 ghi chú mẫu.
- Video ngắn (OBS) quay: mở web → thêm ghi chú → xoá ghi chú → refresh (dữ liệu vẫn còn).
- Ảnh chụp CloudWatch dashboard có số liệu Invocations/Errors/Duration thật.
- (Phần B) Video refresh URL ALB, Instance ID đổi luân phiên.
- Ảnh chụp bảng DynamoDB `Notes` có dữ liệu thật.

## Xử lý lỗi CORS

Nếu console trình duyệt báo lỗi `CORS policy`:
1. Mở lại phần **CORS** của API Gateway, xác nhận `Access-Control-Allow-Origin: *`.
2. Redeploy thủ công stage `$default` rồi thử lại.