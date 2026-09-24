---
title: "Dọn dẹp tài nguyên và kiểm soát chi phí"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Mục tiêu

- Xoá toàn bộ tài nguyên Phần B ngay sau khi kiểm thử, để không phát sinh chi phí liên tục.
- Giữ chi phí của cả workshop ở mức 0 trong phạm vi Free Tier.

## Các bước

### 1. Dọn dẹp Phần B

Xoá theo thứ tự phụ thuộc (tài nguyên phụ thuộc xoá trước):

1. **Auto Scaling Group** `cloudnote-asg`.
2. **Load Balancer** `cloudnote-alb`.
3. **Target Group** `cloudnote-tg`.
4. **Launch Template** `cloudnote-web-template`.
5. **VPC** `cloudnote-vpc`, kèm subnet, route table, Internet Gateway và security group `cloudnote-web-sg`.
6. **Key Pair** `cloudnote-key`.

Sau đó kiểm tra lại từng mục, xác nhận không còn tài nguyên nào của Phần B.

### 2. Kiểm soát chi phí

- **Zero-Spend Budget:** cảnh báo qua email ngay khi phát sinh chi phí. IAM user mặc định không vào được Billing, nên em đăng nhập root đúng một lần để tạo budget rồi đăng xuất.
- **Không dùng NAT Gateway** trong Phần B; EC2 dùng public subnet với auto-assign public IPv4 thay thế.
- **Không nâng cấp lên Paid plan** khi Bedrock, Comprehend, Translate bị chặn; chuyển sang Amazon Polly ([5.6](../5.6-polly-read-note/)).
- DynamoDB chạy chế độ **On-demand**; Polly chỉ nhận tối đa 3000 ký tự mỗi lần đọc (`text[:3000]`).

### 3. Tài nguyên CloudNote còn giữ lại

Phần A vẫn đang chạy để demo: IAM user/group/role, bảng DynamoDB `Notes`, Lambda `notes-api`, HTTP API `notes-http-api`, bucket S3 web tĩnh và bucket log CloudTrail, Cognito User Pool, CloudWatch dashboard và alarm, SNS topic `cloudnote-alerts`, trail `cloudnote-audit-trail`, Zero-Spend Budget.

## Lỗi gặp phải và cách xử lý

| Lỗi | Cách xử lý |
|-----|------------|
| IAM user không xem được Billing | Mặc định AWS chặn IAM user vào Billing → đăng nhập root một lần để tạo Zero-Spend Budget, sau đó đăng xuất |

## Kết quả

- Đã xoá sạch tài nguyên Phần B, không phát sinh chi phí liên tục.
- Zero-Spend Budget đang hoạt động.
