---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Workshop: CloudNote — ứng dụng ghi chú serverless trên AWS

Workshop ghi lại các bước em đã thực hiện để xây dựng dự án tổng hợp **CloudNote**: ứng dụng web ghi chú cá nhân có đăng nhập, chạy trên các dịch vụ serverless của AWS trong phạm vi Free Tier. Frontend tĩnh lưu trên **Amazon S3**, người dùng đăng nhập bằng **Amazon Cognito**, API CRUD chạy trên **API Gateway + AWS Lambda**, dữ liệu lưu trong **Amazon DynamoDB**, tính năng "Đọc ghi chú" dùng **Amazon Polly**, giám sát bằng **Amazon CloudWatch** và kiểm toán bằng **AWS CloudTrail**. Phần B là bài thực hành riêng về **VPC, EC2, ALB và Auto Scaling**, đã dọn dẹp sau khi kiểm thử.

**Region:** `ap-southeast-1` (Singapore)

#### Nội dung

1. [Tổng quan workshop](5.1-workshop-overview/)
2. [IAM: user, role và group](5.2-iam/)
3. [Backend: DynamoDB, Lambda, API Gateway](5.3-backend/)
4. [Frontend trên S3](5.4-frontend/)
5. [Xác thực người dùng với Cognito](5.5-authentication/)
6. [Tính năng "Đọc ghi chú" bằng Amazon Polly](5.6-polly-read-note/)
7. [Giám sát và kiểm toán: CloudWatch, CloudTrail](5.7-monitoring/)
8. [Phần B: VPC, ALB, Auto Scaling](5.8-part-b-web-tier/)
9. [Dọn dẹp tài nguyên và kiểm soát chi phí](5.9-cleanup-cost/)
