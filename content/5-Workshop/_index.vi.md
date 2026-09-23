---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Workshop: CloudNote — Triển khai ứng dụng Serverless

![Ảnh chụp workshop](/images/5-Workshop/image1.png)

Các bước thực hành triển khai đồ án capstone **CloudNote** — ứng dụng **ghi chú serverless** trên AWS Free Tier: frontend tĩnh host trên **S3**, API CRUD với **Lambda + API Gateway**, dữ liệu lưu trong **DynamoDB**, giám sát bằng **CloudWatch** và kiểm toán bằng **CloudTrail**. Phần bonus bổ sung web tier có khả năng mở rộng ngang với **VPC, EC2, ALB và Auto Scaling**. Repository CloudNote đang được chuẩn bị — link sẽ cập nhật sau.

**Region:** `ap-southeast-1`  
**Phạm vi:** Lõi serverless (Phần A): IAM, DynamoDB `Notes`, Lambda `notes-api`, API Gateway HTTP API, S3 static hosting, CloudWatch dashboard/alarm, CloudTrail. Web tier bonus (Phần B): VPC, Launch Template, Target Group, ALB, Auto Scaling.

#### Nội dung

1. [Tổng quan workshop](5.1-Workshop-overview/)
2. [EC2 web tier & Auto Scaling](5.2-EC2-Fleet/)
3. [S3 static app hosting](5.3-S3-Hosting/)
4. [IAM roles & policies](5.4-IAM/)
5. [GitHub OIDC → AWS (xác thực CI)](5.5-GitHub-OIDC/)
6. [Backend DynamoDB, Lambda & API Gateway](5.6-CodeDeploy/)
7. [Giám sát & kiểm toán (CloudWatch, CloudTrail)](5.7-Async-Processing/)
8. [VPC — serverless private networking](5.8-VPC-Networking/)
9. [App demo](5.9-App-Demo/)
10. [Dọn dẹp tài nguyên (có tài liệu)](5.10-Cleanup/)