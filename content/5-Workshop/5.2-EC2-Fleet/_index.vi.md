---
title: "EC2 Web Tier & Auto Scaling"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Mục tiêu

Thiết lập **web tier Phần B**: **EC2 Launch Template** kèm **Auto Scaling Group (ASG)** phía sau Application Load Balancer để web tier CloudNote mở rộng ngang được. Đây là mẫu phục vụ web cổ điển, mở rộng được (VPC, EC2, ELB, Auto Scaling), bổ trợ cho lõi serverless của Phần A.

## Bước 1 — Tạo Launch Template

1. **EC2 → Launch Templates → Create launch template** tại `ap-southeast-1`.
2. **Name:** `cloudnote-web-template`.
3. **AMI:** **Amazon Linux 2023** (Free tier eligible).
4. **Instance type:** `t2.micro` (hoặc `t3.micro` nếu phù hợp).
5. **Key pair:** tạo `cloudnote-key`, tải file `.pem` về máy lưu cẩn thận.

![Tạo launch template](/images/5-Workshop/image4.png)

6. **Network settings → Security groups → Create security group**:
   - Name: `cloudnote-web-sg`
   - Inbound rules: `HTTP (80)` từ `0.0.0.0/0`, `SSH (22)` chỉ từ IP của bạn.
7. **Advanced details → User data** — cài web server và hiển thị Instance ID:

```bash
#!/bin/bash
dnf install -y httpd
systemctl enable httpd
systemctl start httpd
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
echo "<h1>CloudNote Web Tier</h1><p>Phục vụ bởi instance: $INSTANCE_ID</p>" > /var/www/html/index.html
```

8. **Create launch template**.

## Bước 2 — Target Group & Application Load Balancer

1. **EC2 → Target Groups → Create target group**.
2. **Target type:** `Instances` · **Name:** `cloudnote-tg` · **Protocol:** `HTTP:80` · **VPC:** `cloudnote-vpc`.
3. **Health check path:** `/` → tạo group (không đăng ký instance thủ công — ASG tự đăng ký).
4. **EC2 → Load Balancers → Create load balancer → Application Load Balancer**:
   - **Name:** `cloudnote-alb` · **Scheme:** Internet-facing.
   - **VPC:** `cloudnote-vpc`, tick chọn **cả 2 subnet public** (ALB cần ≥ 2 AZ).
   - **Security group:** `cloudnote-web-sg` (hoặc SG riêng cho ALB mở HTTP 80 từ `0.0.0.0/0`).
   - **Listener:** `HTTP:80` → forward tới `cloudnote-tg`.
5. Đợi state = **Active**, copy **DNS name** của ALB.

![Tạo ALB](/images/5-Workshop/image5.png)

## Bước 3 — Tạo Auto Scaling Group

1. **EC2 → Auto Scaling Groups → Create Auto Scaling group**.
2. **Name:** `cloudnote-asg` · **Launch template:** `cloudnote-web-template`.
3. **VPC:** `cloudnote-vpc`, chọn **cả 2 subnet public**.
4. **Load balancing:** attach vào load balancer có sẵn → target group `cloudnote-tg`.
5. **Health checks:** bật thêm **ELB health checks** (không chỉ EC2 health check).
6. **Group size:** Desired = `2`, Minimum = `1`, Maximum = `2`.
7. (Tuỳ chọn) **Target tracking** policy theo average CPU utilization 50%.
8. **Create Auto Scaling group**.

![Tạo ASG](/images/5-Workshop/image5.png)

## Bước 4 — Kiểm tra fleet

1. Đợi 2–3 phút để 2 instance đạt trạng thái **Healthy** trong target group.
2. Mở DNS name của ALB; refresh vài lần — **Instance ID hiển thị phải đổi luân phiên** (cân bằng tải hoạt động).
3. Đây là **sản phẩm demo thứ hai** của capstone, dễ quay video cho báo cáo.

## Kết quả mong đợi

- Instance web tier lặp lại được từ một launch template
- ASG giữ 1–2 instance healthy và mở rộng theo tải
- ALB phân phối traffic HTTP trên cả 2 AZ

## Xử lý sự cố

| Vấn đề | Kiểm tra |
|---|---|
| Target **Unhealthy** | Security group cho phép HTTP 80 từ `0.0.0.0/0`; User Data chạy được (`systemctl status httpd`) |
| ASG không đạt được desired capacity | Quota vCPU, loại instance có sẵn, subnet sai |
| DNS ALB không đổi | Cache trình duyệt; kiểm tra cả 2 instance in service |