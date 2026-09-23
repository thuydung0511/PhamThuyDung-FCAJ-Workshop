---
title: "Cleanup"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

Sau khi ghi lại đầy đủ ảnh chụp và video demo của capstone, em tháo dỡ toàn bộ stack **CloudNote** theo đúng thứ tự phụ thuộc để giữ tài khoản gọn gàng và tránh phát sinh phí. Với từng tài nguyên AWS, em mở trang console tương ứng và chọn delete/terminate.

**Dọn Phần B trước (ưu tiên cao nhất — ALB tính phí theo giờ):**

1. **EC2 → Auto Scaling Groups** → chọn `cloudnote-asg` → **Delete** (instance bên trong bị terminate tự động).
2. **EC2 → Load Balancers** → chọn `cloudnote-alb` → **Actions → Delete**.
3. **EC2 → Target Groups** → chọn `cloudnote-tg` → **Delete**.
4. **EC2 → Launch Templates** → chọn `cloudnote-web-template` → **Delete**.
5. **EC2 → Instances** → terminate các instance độc lập còn sót lại nếu có.
6. **VPC → Your VPCs** → chọn `cloudnote-vpc` → **Actions → Delete VPC** (wizard tự xoá subnet, route table, IGW).
7. **EC2 → Key Pairs** → xoá `cloudnote-key` nếu không còn dùng.

**Dọn Phần A:**

8. **CloudTrail → Trails** → `cloudnote-audit-trail` → **Delete** (có thể giữ lại — management events của trail đầu tiên miễn phí).
9. **CloudWatch** → xoá alarm `notes-api-error-alarm`; xoá dashboard `CloudNote-Dashboard`.
10. **API Gateway** → `notes-http-api` → **Delete**.
11. **Lambda** → `notes-api` → **Delete**.
12. **IAM → Roles** → xoá `LambdaNotesExecutionRole`.
13. **DynamoDB** → bảng `Notes` → **Delete table**.
14. **S3** → **Empty** bucket `cloudnote-app-0205568-2026` (bắt buộc trước khi xoá) → **Delete bucket**; lặp lại với bucket log CloudTrail nếu đã xoá trail ở bước 8.
15. (Tuỳ chọn) **IAM** → xoá user thực hành `cloudnote-dev` hoặc giữ lại cho các workshop tiếp theo.

**Kiểm tra cuối cùng:** mở **AWS Billing → Bills / Cost Explorer** xác nhận không còn tài nguyên đang chạy phát sinh phí (đặc biệt điểm EC2 và ELB).