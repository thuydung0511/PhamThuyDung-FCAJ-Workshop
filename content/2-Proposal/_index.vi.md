---
title: "Bản đề xuất"
date: 2026-08-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# CloudNote — Ứng dụng ghi chú Serverless trên AWS

## Ứng dụng tham chiếu Serverless full-stack, tối ưu chi phí trên AWS

---

### 1. Tóm tắt điều hành

**CloudNote** là ứng dụng ghi chú full-stack chạy hoàn toàn trên kiến trúc **serverless** của AWS, được xây dựng để hoàn tất mục tiêu Capstone Project trong chương trình thực tập *First Cloud AI Journey (FCAJ)* tại Công ty TNHH Amazon Web Services Việt Nam. Người dùng mở một trang web tĩnh (host trên **S3**), đăng ký và đăng nhập bằng **Amazon Cognito**, rồi xem, thêm, sửa, xoá ghi chú trên trình duyệt và nghe ghi chú được **Amazon Polly** đọc thành tiếng. Mỗi người chỉ xem và thao tác được trên ghi chú của chính mình. Toàn bộ dữ liệu được lưu và xử lý serverless — không cần quản lý máy chủ.

Dự án theo nguyên tắc xuyên suốt: *serverless cho mọi thứ, làm đến đâu hiểu đến đó, và giữ chi phí ở mức tối thiểu.* Lõi capstone bao phủ IAM, DynamoDB, Lambda, API Gateway, S3 static hosting, xác thực Cognito, Polly, giám sát CloudWatch kèm cảnh báo email qua SNS và kiểm toán CloudTrail — trong khi web tier bonus (Phần B) minh hoạ khả năng mở rộng ngang với VPC, EC2 Launch Template, Application Load Balancer và Auto Scaling.

---

### 2. Tuyên bố vấn đề

#### Vấn đề là gì?

Xây dựng và chứng minh một ứng dụng "thật" trên cloud đòi hỏi nắm nhiều thành phần: quản trị danh tính và truy cập, kho dữ liệu, lớp compute, tầng API, hosting frontend, giám sát và kiểm toán. Người mới học thường chỉ dừng ở ví dụ đơn giản hoặc nhảy thẳng sang container/Kubernetes mà chưa nắm vững nền tảng.

#### Giải pháp

CloudNote đơn giản về phạm vi nhưng đầy đủ trong vòng đời vận hành. Một **Lambda** duy nhất xử lý CRUD và thao tác "Đọc ghi chú" qua **API Gateway HTTP API**, dữ liệu nằm trên bảng **DynamoDB**. Frontend tĩnh trên **S3** gọi API. **Cognito** xử lý đăng ký, đăng nhập và **JWT Authorizer** trên API Gateway chỉ cho request có token hợp lệ đi qua. **Polly** chuyển ghi chú thành giọng nói. **CloudWatch**, **SNS** và **CloudTrail** đảm bảo quan sát, cảnh báo và kiểm toán. Web tier bonus (VPC + EC2 + ALB + Auto Scaling) bổ sung mẫu phục vụ web theo hướng mở rộng ngang cổ điển.

Lợi ích:

- **Lõi serverless hoàn toàn**: không quản lý EC2 cho ứng dụng chính; Lambda và DynamoDB tự mở rộng.
- **Chi phí thấp**: tài khoản dùng Free account plan với credit; toàn bộ dự án dùng $3.84 trên $200 credit.
- **Đủ kiến thức nền tảng**: IAM least-privilege, mô hình dữ liệu NoSQL, thiết kế API, CORS, xác thực JWT, cách ly dữ liệu theo người dùng, giám sát, kiểm toán, mở rộng ngang.
- **Blueprint tái sử dụng**: áp dụng cho mọi ứng dụng API nhỏ (to-do list, khảo sát, blog backend...).

---

### 3. Kiến trúc giải pháp

Luồng chính hoàn toàn serverless; lab bonus bổ sung web tier mở rộng ngang truyền thống.

![Kiến trúc Serverless CloudNote](/images/5-Workshop/architecture-cloudnote.png)

#### Luồng chính (Phần A — Lõi Serverless)

1. Người dùng mở URL website tĩnh trên **S3** → nhận về `index.html` (HTML, CSS và JavaScript trong một file).
2. Người dùng đăng ký / đăng nhập bằng **Cognito User Pool** và nhận JWT.
3. Trình duyệt gọi **API Gateway HTTP API** (`/notes`) kèm header `Authorization: Bearer <JWT>`.
4. **JWT Authorizer** kiểm tra token; request hợp lệ được chuyển tới **Lambda** `notes-api` (Python 3.12), request không có token hợp lệ nhận **401**.
5. **Lambda** lấy claim `sub` làm `userId` và dùng IAM Role (`LambdaNotesExecutionRole`) để đọc/ghi bảng **DynamoDB** `Notes`, luôn lọc theo `userId`; sửa, xoá hoặc đọc ghi chú của người khác trả về **403**.
6. Với "Đọc ghi chú", Lambda gọi **Amazon Polly** (engine Standard, giọng Joanna) và trả về MP3 để trình duyệt phát.
7. **CloudWatch** ghi log và metric, alarm gửi email qua **SNS** khi Lambda có lỗi; **CloudTrail** ghi lại các lời gọi API trong tài khoản phục vụ kiểm toán.

#### Luồng bonus (Phần B — Web tier)

- **VPC** tuỳ chỉnh với 2 subnet public trên 2 Availability Zone.
- **EC2 Launch Template** (Amazon Linux 2023, t3.micro) dùng User Data để phục vụ trang hiển thị Instance ID.
- **Target Group + Application Load Balancer (ALB)** và **Auto Scaling Group** giữ 1–2 instance; refresh DNS name của ALB thấy Instance ID đổi luân phiên.
- Toàn bộ tài nguyên Phần B được xoá sau khi kiểm thử.

#### Dịch vụ AWS sử dụng

- **IAM**: user thực hành `cloudnote-dev`; group `cloudnote-network-group`; execution role `LambdaNotesExecutionRole` + inline policy `NotesTableAccess` và `PollySynthesizeSpeech`.
- **Amazon DynamoDB**: bảng `Notes` (partition key `noteId`, on-demand capacity); mỗi ghi chú lưu `userId` của chủ sở hữu.
- **AWS Lambda**: `notes-api` — một function duy nhất xử lý GET/POST/PUT/DELETE và "Đọc ghi chú", lọc dữ liệu theo `userId`.
- **Amazon API Gateway**: HTTP API `notes-http-api` với 5 route — `GET/POST /notes`, `PUT/DELETE /notes/{noteId}` và `POST /notes/{noteId}/speak` — tất cả được bảo vệ bằng JWT Authorizer.
- **Amazon S3**: bucket `cloudnote-app-0205568-2026` — static website hosting + bucket policy public read.
- **Amazon Cognito**: User Pool (đăng nhập bằng email) và App Client loại Single-page application.
- **Amazon Polly**: engine Standard, giọng Joanna, định dạng MP3.
- **Amazon CloudWatch**: dashboard `CloudNote-Dashboard`; alarm `notes-api-error-alarm`.
- **Amazon SNS**: topic `cloudnote-alerts` gửi email cảnh báo.
- **AWS CloudTrail**: trail `cloudnote-audit-trail` (multi-region, management events).
- **VPC / EC2 / ELB / Auto Scaling** (Phần B): `cloudnote-vpc`, `cloudnote-web-template`, `cloudnote-tg`, `cloudnote-alb`, `cloudnote-asg`.

---

### 4. Triển khai kỹ thuật

#### Các bước triển khai (Phần A)

1. **A0 — IAM user**: tạo `cloudnote-dev` có console access, gắn 7 managed policy FullAccess phục vụ thực hành (S3, DynamoDB, Lambda, API Gateway, IAM, CloudWatch, CloudTrail); dùng user này thay cho root.
2. **A1 — DynamoDB**: tạo bảng `Notes`, partition key `noteId` (String), capacity **On-demand**.
3. **A2 — IAM Role**: tạo `LambdaNotesExecutionRole` với `AWSLambdaBasicExecutionRole` + inline `NotesTableAccess` (CRUD DynamoDB giới hạn ARN bảng `Notes`).
4. **A3 — Lambda**: tạo `notes-api` (Python 3.12), gán role, implement CRUD, chạy Test event và kiểm tra item trong DynamoDB.
5. **A4 — API Gateway**: tạo HTTP API `notes-http-api`, thêm 4 route CRUD tới `notes-api`, bật CORS, ghi lại Invoke URL.
6. **A5 — Kiểm thử API**: test đủ 4 method và preflight CORS bằng PowerShell (`Invoke-RestMethod`); không dùng được CloudShell vì tài khoản mới đang được xác minh.
7. **A6 — Hosting S3**: tạo bucket `cloudnote-app-0205568-2026`, bật static website hosting, gắn bucket policy public read, upload `index.html` (frontend gọi Invoke URL).
8. **A7 — Kiểm thử E2E**: mở website endpoint, tạo và xoá ghi chú trên trình duyệt, tải lại trang và đối chiếu với DynamoDB.
9. **A8 — CloudWatch**: tạo dashboard với Lambda `Invocations/Errors/Duration` + DynamoDB `Consumed*Capacity`, tạo alarm trên Lambda Errors, gửi cảnh báo email qua SNS topic `cloudnote-alerts`.
10. **A9 — CloudTrail**: tạo trail `cloudnote-audit-trail` (multi-region, management events, bucket S3 mới lưu log).
11. **Xác thực**: tạo Cognito User Pool và App Client, gắn JWT Authorizer cho mọi route, thêm đăng ký/đăng nhập vào frontend và lọc dữ liệu theo `userId` trong Lambda (PUT/DELETE kiểm tra chủ sở hữu).
12. **"Đọc ghi chú"**: thêm inline policy `PollySynthesizeSpeech` (chỉ `polly:SynthesizeSpeech`) cho role của Lambda, thêm route `POST /notes/{noteId}/speak` (tổng 5 route), Lambda gọi Polly sau khi kiểm tra ghi chú thuộc về người gọi, thêm nút "🔊 Đọc" vào frontend.

#### Các bước triển khai (Phần B — tuỳ chọn)

B1 VPC tuỳ chỉnh (2 AZ, 2 subnet public, không NAT) → B2 launch template `cloudnote-web-template` (t3.micro) → B3 target group `cloudnote-tg` + ALB `cloudnote-alb` → B4 ASG `cloudnote-asg` (desired 2, min 1, max 2, ELB health checks) → B5 kiểm chứng cân bằng tải đổi Instance ID → dọn dẹp toàn bộ tài nguyên Phần B.

B6 (nâng cao – tuỳ chọn) Docker + ECS Fargate: **chưa thực hiện**.

#### Yêu cầu kỹ thuật

- Chỉ dùng AWS Console (không bắt buộc CLI); region **ap-southeast-1 (Singapore)**.
- Test API bằng PowerShell trên máy cá nhân.

#### Các biện pháp bảo mật đã thực hiện

- Đã bật **MFA** (ứng dụng Authenticator) cho tài khoản root; tài khoản root không có access key.
- Làm việc hằng ngày bằng IAM user `cloudnote-dev`, không dùng root.
- Role của Lambda chỉ có đúng quyền cần thiết (least privilege): ghi log, 6 thao tác trên bảng `Notes` và `polly:SynthesizeSpeech`.
- Mọi route API đều yêu cầu JWT hợp lệ từ Cognito; Lambda kiểm tra quyền sở hữu ghi chú theo `userId`.
- Frontend thoát ký tự HTML trước khi hiển thị nội dung ghi chú (chống XSS) và chỉ giữ token trong bộ nhớ, không lưu `localStorage`.

---

### 5. Timeline & Milestone

- **Tuần 1 (01/08 – 14/08)**: Tổng quan kiến trúc AWS; tạo tài khoản AWS; tìm hiểu Lab 000001 (AWS Free Tier).
- **Tuần 2 (15/08 – 28/08)**: Dịch vụ cốt lõi EC2, S3, IAM; Lab 000004; hoàn thành 5 nhiệm vụ "Explore AWS" của Lab 000001.
- **Tuần 3 (29/08 – 11/09)**: Mạng AWS (VPC, subnet, Internet Gateway); chọn ý tưởng CloudNote.
- **Tuần 4 (12/09 – 19/09)**: Lambda, serverless, CloudWatch, CloudTrail; thiết kế kiến trúc CloudNote; bắt đầu Phần A (A0–A2).
- **Tuần 5 (20/09 – 27/09)**: Hoàn thành Phần A (A3–A9), web tier Phần B và dọn dẹp, xác thực Cognito, "Đọc ghi chú" bằng Polly, website báo cáo.

---

### 6. Ước tính ngân sách

Tài khoản AWS dùng **Free account plan** với credit: $100 khi tạo tài khoản và thêm $100 khi hoàn thành các nhiệm vụ "Explore AWS", tổng cộng $200. Region `ap-southeast-1`.

| Dịch vụ | Mức sử dụng workshop | Chi phí |
|---|---|---|
| S3 | 1 bucket web + 1 bucket log CloudTrail | Credit bù |
| DynamoDB (on-demand) | 1 bảng, vài item | Credit bù |
| AWS Lambda | 1 function, số lần gọi thấp | Credit bù |
| API Gateway (HTTP API) | 1 API, 5 route | Credit bù |
| Cognito | 1 User Pool, vài user | Credit bù |
| Polly | Engine Standard, tối đa 3000 ký tự mỗi request | Credit bù |
| CloudWatch + SNS | 1 dashboard, 1 alarm, 1 topic email | Credit bù |
| CloudTrail | 1 trail | Credit bù |
| EC2 + ALB (Phần B) | 2 instance t3.micro + 1 ALB, xoá sau khi kiểm thử | Credit bù |
| **Tổng thực tế** | | **Đã dùng $3.84 trên $200 credit; thanh toán $0.00** |

Kiểm soát ngân sách: **Zero-Spend Budget** trong AWS Billing, không dùng NAT Gateway ở Phần B, xoá Phần B ngay sau khi kiểm thử và không nâng cấp lên Paid plan.

---

### 7. Đánh giá rủi ro

| Rủi ro | Tác động | Xác suất | Cách giảm thiểu |
| --- | --- | --- | --- |
| Phát sinh chi phí ngoài ý muốn (ALB) | Trung bình | Trung bình | Zero-spend budget; xoá Phần B ngay sau khi kiểm thử; theo đúng thứ tự cleanup trong workshop |
| Lỗi CORS trên trình duyệt | Trung bình | Cao | Cấu hình CORS API Gateway (gồm cả header `Authorization`); kiểm chứng với frontend |
| Lambda trả lỗi 500 | Trung bình | Trung bình | Xem CloudWatch Logs (`/aws/lambda/notes-api`); kiểm tra ARN trong `NotesTableAccess` |
| Người dùng xem hoặc sửa ghi chú của người khác | Cao | Trung bình | JWT Authorizer trên mọi route; Lambda lọc theo `userId` và kiểm tra chủ sở hữu (403) |
| Dịch vụ AI bị chặn trên Free account plan | Trung bình | Cao | Dùng Amazon Polly, dịch vụ dùng được trên Free account plan |
| ASG không đạt tới target | Trung bình | Thấp | Kiểm tra SG mở HTTP 80 từ `0.0.0.0/0`; xem trạng thái health của ALB |

---

### 8. Kết quả mong đợi

- Một **ứng dụng serverless full-stack hoạt động thật** (CloudNote) chạy trên AWS kèm demo trên trình duyệt, đăng nhập bằng Cognito, dữ liệu riêng cho từng người dùng và tính năng "Đọc ghi chú".
- Hiểu sâu IAM least-privilege, DynamoDB, Lambda + API Gateway, S3 static hosting, Cognito, Polly, CloudWatch và CloudTrail.
- Demo bonus thứ hai: ALB cân bằng tải qua 2 instance EC2 trong VPC tuỳ chỉnh.
- Bộ ảnh chụp và báo cáo đầy đủ phục vụ deliverable thực tập và tự đánh giá.
