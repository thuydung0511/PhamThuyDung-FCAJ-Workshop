---
title: "Phần B: VPC, ALB, Auto Scaling"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Mục tiêu

Bài thực hành riêng, không thuộc kiến trúc CloudNote: dựng một web tier gồm 2 máy EC2 trong 2 Availability Zone, đặt sau Application Load Balancer và do Auto Scaling Group quản lý, rồi kiểm tra cân bằng tải. Toàn bộ tài nguyên đã được xoá sau khi kiểm thử ([5.9](../5.9-cleanup-cost/)).

## Các bước

### 0. Chuẩn bị

- Kiểm tra Zero-Spend Budget và email cảnh báo còn hoạt động.
- Cấp quyền VPC, EC2, ELB cho `cloudnote-dev` qua IAM group `cloudnote-network-group` ([5.2](../5.2-iam/)).

### 1. VPC (B1)

1. **VPC → Create VPC → VPC and more**, tên `cloudnote-vpc`, CIDR `10.0.0.0/16`.
2. 2 Availability Zone, **2 public subnet, 0 private subnet**, **không dùng NAT Gateway**.
3. Kết quả: subnet `...-public1-ap-southeast-1a`, `...-public2-ap-southeast-1b`, Internet Gateway `cloudnote-igw`, route table `cloudnote-rtb-public`.
4. Bật **Auto-assign public IPv4** cho cả 2 subnet: không có NAT Gateway nên EC2 cần IP public để tải gói cài đặt.

### 2. Launch Template (B2)

1. **EC2 → Launch Templates → Create**, tên `cloudnote-web-template`.
2. AMI **Amazon Linux 2023**, instance type **t3.micro**, key pair `cloudnote-key`.
3. Security group `cloudnote-web-sg`:
   - HTTP 80 cho mọi nguồn.
   - SSH 22 chỉ cho IP cá nhân.
4. **User Data** cài Apache và hiển thị Instance ID của máy; em chỉnh thêm:
   - Lấy token **IMDSv2** trước khi đọc metadata Instance ID.
   - Khai báo **charset UTF-8** để trang hiển thị đúng tiếng Việt.

### 3. Target Group và ALB (B3)

1. Target Group `cloudnote-tg`: HTTP:80, health check `/`.
2. ALB `cloudnote-alb`: Internet-facing, chọn 2 public subnet, security group `cloudnote-web-sg`.
3. Listener HTTP:80 forward về `cloudnote-tg`.

### 4. Auto Scaling Group (B4)

1. `cloudnote-asg` dùng Launch Template `cloudnote-web-template`, 2 public subnet.
2. **Attach to an existing load balancer**, chọn Target Group `cloudnote-tg`.
3. Desired **2** / Min **1** / Max **2**; bật cả EC2 và **ELB health check**.

### 5. Kiểm tra cân bằng tải (B5)

Mở `http://<ALB_DNS_NAME>` và tải lại trang nhiều lần, quan sát Instance ID hiển thị.

## Lỗi gặp phải và cách xử lý

| Lỗi | Cách xử lý |
|-----|------------|
| Vượt giới hạn 10 managed policy gắn trực tiếp cho IAM user | Cấp quyền qua IAM group `cloudnote-network-group` |
| Wizard VPC mặc định 0 public / 2 private subnet | Chỉnh lại thành 2 public / 0 private |
| ALB bị chọn thêm security group `default` | Bỏ chọn, chỉ giữ `cloudnote-web-sg` |
| ASG ban đầu để "No load balancer" | Sửa thành "Attach to an existing load balancer", chọn đúng Target Group, bật ELB health check |
| Trình duyệt báo time-out khi mở DNS của ALB dù mọi target Healthy | Kiểm tra bằng `nslookup` và `curl` trong Command Prompt → hạ tầng AWS bình thường, lỗi do cache/tiện ích trình duyệt → mở bằng cửa sổ ẩn danh thì chạy |

## Kết quả kiểm thử

- 2 Availability Zone, mỗi AZ một public subnet.
- ASG có 2/2 instance **Healthy**; truy cập qua ALB trả **HTTP 200**, hiển thị đúng Instance ID.
- Tải lại trang: Instance ID luân phiên giữa 2 máy → cân bằng tải hoạt động.
