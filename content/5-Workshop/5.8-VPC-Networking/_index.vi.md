---
title: "VPC & mạng web tier"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Mục tiêu

Tạo **VPC tuỳ chỉnh** cho web tier Phần B (`cloudnote-vpc`) với 2 subnet public trên 2 Availability Zone và Internet Gateway — nền tảng mạng cho fleet EC2 của ALB + ASG. (Lõi serverless Phần A không cần VPC.)

## Bước 1 — Tạo VPC bằng wizard

1. **VPC → Create VPC → VPC and more** (wizard tự tạo đầy đủ).
2. **Name tag:** `cloudnote-vpc`.
3. **IPv4 CIDR:** `10.0.0.0/16`.
4. **Number of Availability Zones:** `2`.
5. **Number of public subnets:** `2` · **private subnets:** `0` (tránh phí NAT).
6. **NAT gateways:** **None** · **VPC endpoints:** **None**.
7. **Create VPC**.

![Tạo VPC](/images/5-Workshop/5.8-VPC-Networking/vpc.png)

## Bước 2 — Xác minh các thành phần mạng

1. **Your VPCs:** có `cloudnote-vpc`.
2. **Subnets:** 2 subnet public ở 2 AZ khác nhau.
3. **Internet Gateways:** 1 IGW gắn với VPC.

![Subnets](/images/5-Workshop/5.8-VPC-Networking/subnets.png)

## Bước 3 — Dùng cho web tier

- Đặt Load Balancer + instance ASG trong cả 2 subnet public ([5.2](5.2-EC2-Fleet/)); ALB cần ≥ 2 AZ.
- Security group `cloudnote-web-sg` cho phép HTTP `80` từ `0.0.0.0/0` và SSH `22` từ IP của bạn.

![Security groups](/images/5-Workshop/5.8-VPC-Networking/security-groups.png)

## Kết quả mong đợi

- Mạng riêng, cách ly cho web tier bonus
- Hai AZ đảm bảo tính khả dụng cao
- Truy cập internet qua IGW đã gắn

## Xử lý sự cố

| Vấn đề | Kiểm tra |
|--------|----------|
| Target ALB unhealthy | Instance và ALB phải cùng VPC; SG cho phép HTTP 80 |
| Instance không có internet | IGW gắn với VPC; route table subnet có default route qua IGW |
| ASG không launch được | Subnet chọn phải thuộc `cloudnote-vpc` |