---
title: "Bản đề xuất"
date: 2026-08-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# CloudNote — Ứng dụng ghi chú Serverless trên AWS

## Ứng dụng tham chiếu Serverless full-stack, tối ưu chi phí, xây dựng trên AWS Free Tier

---

### 1. Tóm tắt điều hành

**CloudNote** là ứng dụng ghi chú full-stack chạy hoàn toàn trên kiến trúc **serverless** của AWS, được xây dựng để hoàn tất mục tiêu Capstone Project trong chương trình thực tập *First Cloud AI Journey (FCAJ)* tại Công ty TNHH Amazon Web Services Việt Nam. Người dùng mở một trang web tĩnh (host trên **S3**), xem danh sách ghi chú, đồng thời thêm, sửa, xoá ghi chú hoàn toàn qua trình duyệt. Toàn bộ dữ liệu được lưu và xử lý serverless — không cần quản lý máy chủ.

Dự án theo nguyên tắc xuyên suốt: *serverless cho mọi thứ, làm đến đâu hiểu đến đó, và luôn nằm trong AWS Free Tier.* Lõi capstone bao phủ đầy đủ vòng đời — IAM, DynamoDB, Lambda, API Gateway, S3 static hosting, giám sát CloudWatch và kiểm toán CloudTrail — trong khi web tier bonus (Phần B) minh hoạ khả năng mở rộng ngang với VPC, EC2 Launch Template, Application Load Balancer và Auto Scaling.

---

### 2. Tuyên bố vấn đề

#### Vấn đề là gì?

Xây dựng và chứng minh một ứng dụng "thật" trên cloud đòi hỏi nắm nhiều thành phần: quản trị danh tính và truy cập, kho dữ liệu, lớp compute, tầng API, hosting frontend, giám sát và kiểm toán. Người mới học thường chỉ dừng ở ví dụ đơn giản hoặc nhảy thẳng sang container/Kubernetes mà chưa nắm vững nền tảng.

#### Giải pháp

CloudNote đơn giản về phạm vi nhưng đầy đủ trong vòng đời vận hành. Một **Lambda** duy nhất xử lý CRUD qua **API Gateway HTTP API**, dữ liệu nằm trên bảng **DynamoDB**. Frontend tĩnh trên **S3** gọi API. **CloudWatch** và **CloudTrail** đảm bảo quan sát và kiểm toán. Web tier bonus (VPC + EC2 + ALB + Auto Scaling) bổ sung mẫu phục vụ web theo hướng mở rộng ngang cổ điển.

Lợi ích:

- **Lõi serverless hoàn toàn**: không quản lý EC2 cho ứng dụng chính; Lambda và DynamoDB tự mở rộng.
- **Thân thiện Free Tier**: toàn bộ Phần A = **$0** trong hạn mức; Phần B chỉ ~0,07–0,15 USD cho ~3 giờ chạy ALB.
- **Đủ kiến thức nền tảng**: IAM least-privilege, mô hình dữ liệu NoSQL, thiết kế API, CORS, giám sát, kiểm toán, mở rộng ngang.
- **Blueprint tái sử dụng**: áp dụng cho mọi ứng dụng API nhỏ (to-do list, khảo sát, blog backend...).

---

### 3. Kiến trúc giải pháp

Luồng chính hoàn toàn serverless; lab bonus bổ sung web tier mở rộng ngang truyền thống.

![Kiến trúc Serverless CloudNote](/images/2-Proposal/architecture.png)

#### Luồng chính (Phần A — Lõi Serverless)

1. Người dùng mở URL website tĩnh trên **S3** → nhận về `index.html` + `app.js`.
2. `app.js` gọi API qua `fetch` tới endpoint **API Gateway HTTP API** (`/notes`).
3. **API Gateway** định tuyến request tới **Lambda** `notes-api` (Python 3.12).
4. **Lambda** dùng IAM Role được gán (`LambdaNotesExecutionRole`) để đọc/ghi bảng **DynamoDB** `Notes`.
5. Kết quả trả ngược qua API Gateway và hiển thị trên trình duyệt.
6. **CloudWatch** ghi log và cảnh báo lỗi; **CloudTrail** ghi lại mọi lời gọi API trong tài khoản phục vụ kiểm toán.

#### Luồng bonus (Phần B — Web tier)

- **VPC** tuỳ chỉnh với 2 subnet public trên 2 Availability Zone.
- **EC2 Launch Template** (Amazon Linux 2023, t2.micro) dùng User Data để phục vụ trang hiển thị Instance ID.
- **Target Group + Application Load Balancer (ALB)** và **Auto Scaling Group** giữ 1–2 instance; refresh DNS name của ALB thấy Instance ID đổi luân phiên.

#### Dịch vụ AWS sử dụng

- **IAM**: user thực hành `cloudnote-dev`; execution role `LambdaNotesExecutionRole` + inline policy `NotesTableAccess`.
- **Amazon DynamoDB**: bảng `Notes` (partition key `noteId`, on-demand capacity).
- **AWS Lambda**: `notes-api` — một function CRUD duy nhất (GET/POST/PUT/DELETE).
- **Amazon API Gateway**: HTTP API `notes-http-api` với route `GET/POST /notes` và `PUT/DELETE /notes/{noteId}`.
- **Amazon S3**: bucket `cloudnote-app-0205568-2026` — static website hosting + bucket policy public read.
- **Amazon CloudWatch**: dashboard `CloudNote-Dashboard`; alarm `notes-api-error-alarm`.
- **AWS CloudTrail**: trail `cloudnote-audit-trail` (management events).
- **VPC / EC2 / ELB / Auto Scaling** (Phần B): `cloudnote-vpc`, `cloudnote-web-template`, `cloudnote-tg`, `cloudnote-alb`, `cloudnote-asg`.

---

### 4. Triển khai kỹ thuật

#### Các bước triển khai (Phần A)

1. **A0 — IAM user**: tạo `cloudnote-dev` có console access; gắn policy thực hành; không bao giờ dùng root.
2. **A1 — DynamoDB**: tạo bảng `Notes`, partition key `noteId` (String), capacity **On-demand**.
3. **A2 — IAM Role**: tạo `LambdaNotesExecutionRole` với `AWSLambdaBasicExecutionRole` + inline `NotesTableAccess` (CRUD DynamoDB giới hạn ARN bảng `Notes`).
4. **A3 — Lambda**: tạo `notes-api` (Python 3.12, x86_64), gán role, implement CRUD, test POST ngay trong console.
5. **A4 — API Gateway**: tạo HTTP API `notes-http-api`, thêm 4 route tới `notes-api`, bật CORS, ghi lại Invoke URL.
6. **A5 — Kiểm thử API**: xác nhận `GET /notes` trả về JSON; test POST bằng trình duyệt/CloudShell curl.
7. **A6 — Hosting S3**: tạo bucket `cloudnote-app-0205568-2026`, bật static website hosting, gắn bucket policy public read, upload `index.html` (frontend gọi Invoke URL).
8. **A7 — Kiểm thử E2E**: mở website endpoint, tạo và xoá ghi chú trên trình duyệt.
9. **A8 — CloudWatch**: tạo dashboard với Lambda `Invocations/Errors/Duration` + DynamoDB `Consumed*Capacity`, tạo alarm trên Lambda Errors.
10. **A9 — CloudTrail**: tạo trail `cloudnote-audit-trail` (management events, bucket S3 mới lưu log).

#### Các bước triển khai (Phần B — tuỳ chọn)

B1 VPC tuỳ chỉnh (2 AZ, 2 subnet public, không NAT) → B2 launch template `cloudnote-web-template` → B3 target group `cloudnote-tg` + ALB `cloudnote-alb` → B4 ASG `cloudnote-asg` (desired 2, min 1, max 2, ELB health checks) → B5 kiểm chứng cân bằng tải đổi Instance ID → B6 (nâng cao – tuỳ chọn) Docker + ECS Fargate.

#### Yêu cầu kỹ thuật

- Chỉ dùng AWS Console (không bắt buộc CLI); region **ap-southeast-1 (Singapore)**.
- Bật MFA cho tài khoản root.
- Dùng tài khoản mới hoặc ít traffic để nằm trong hạn mức Free Tier.

---

### 5. Timeline & Milestone

- **Tuần 1–2**: Tạo tài khoản AWS, MFA, IAM, lab EC2/VPC (nền tảng cho Phần B).
- **Tuần 3**: Khám phá dịch vụ, deploy ứng dụng trên S3 và EC2.
- **Tuần 4**: Lưu trữ (S3, DynamoDB, EBS).
- **Tuần 5**: Giám sát CloudWatch + AWS CLI.
- **Tuần 6**: Capstone Phần A — DynamoDB, Lambda, IAM role, backend CRUD.
- **Tuần 7**: API Gateway + CORS, frontend S3, demo end-to-end, CloudWatch + CloudTrail, (tuỳ chọn) Phần B web tier, báo cáo cuối.

---

### 6. Ước tính ngân sách

Giả định: tài khoản AWS **mới** (Free Tier 12 tháng), region `ap-southeast-1`; Phần A hoàn tất trong vài ngày; Phần B chạy ~3 giờ rồi dọn dẹp ngay.

| Dịch vụ | Hạn mức Free Tier | Mức sử dụng workshop | Chi phí ước tính |
|---|---|---|---|
| S3 (Phần A) | 5 GB + 20K GET/tháng | 1 bucket nhỏ | **$0** |
| DynamoDB (on-demand) | 25 GB + 2,5M đọc/tháng (Always Free) | vài chục item | **$0** |
| AWS Lambda | 1M request + 400K GB-giây/tháng | vài trăm lần gọi | **$0** |
| API Gateway (HTTP API) | 1M request/tháng (12 tháng đầu) | vài trăm request | **$0** |
| CloudWatch | 10 custom metrics, 3 dashboards | 1 dashboard + alarm | **$0** |
| CloudTrail | Management events trail đầu miễn phí | 1 trail | **$0** |
| EC2 t2/t3.micro (Phần B) | 750 giờ/tháng | 2 × 3 giờ = 6 giờ | **$0** |
| ALB (Phần B) | **Không có Free Tier** | ~3 giờ | **≈ $0,07–0,15** |
| Data transfer out | 100 GB/tháng | vài MB | **$0** |
| **Tổng** | | | **≈ 0–0,20 USD** |

Kiểm soát ngân sách: tạo **Zero spend budget** trong AWS Billing trước khi bắt đầu Phần B, dọn Phần B ngay trong cùng buổi, theo dõi **Cost Explorer** hàng ngày tuần thực hiện.

---

### 7. Đánh giá rủi ro

| Rủi ro | Tác động | Xác suất | Cách giảm thiểu |
| --- | --- | --- | --- |
| Phát sinh chi phí ngoài ý muốn (ALB) | Trung bình | Trung bình | Zero-spend budget; dọn Phần B cùng buổi; theo đúng thứ tự cleanup trong workshop |
| Lỗi CORS trên trình duyệt | Trung bình | Cao | Cấu hình CORS API Gateway; redeploy stage `$default`; kiểm chứng với frontend |
| Lambda trả lỗi 500 | Trung bình | Trung bình | Xem CloudWatch Logs (`/aws/lambda/notes-api`); kiểm tra ARN trong `NotesTableAccess` |
| ASG không đạt tới target | Trung bình | Thấp | Kiểm tra SG mở HTTP 80 từ `0.0.0.0/0`; xem trạng thái health của ALB |

---

### 8. Kết quả mong đợi

- Một **ứng dụng serverless full-stack hoạt động thật** (CloudNote) chạy trên AWS Free Tier kèm demo trên trình duyệt.
- Hiểu sâu IAM least-privilege, DynamoDB, Lambda + API Gateway, S3 static hosting, CloudWatch và CloudTrail.
- Demo bonus thứ hai: ALB cân bằng tải qua 2 instance EC2 trong VPC tuỳ chỉnh.
- Bộ ảnh chụp/video/báo cáo đầy đủ phục vụ deliverable thực tập và tự đánh giá.