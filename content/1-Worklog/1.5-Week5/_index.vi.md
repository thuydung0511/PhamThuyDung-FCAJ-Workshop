---
title: "Worklog Tuần 5"
date: 2026-09-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

* Tiếp tục triển khai dự án tổng hợp **CloudNote** trên AWS (đã bắt đầu A0–A2 ở tuần 4) theo kiến trúc đã thiết kế ở tuần 4.
* Bổ sung xác thực người dùng, cách ly dữ liệu theo người dùng và một tính năng AI.
* Thực hành riêng Phần B: VPC, ALB, Auto Scaling, sau đó dọn dẹp tài nguyên.
* Đưa website báo cáo thực tập lên GitHub Pages.

**Thời gian:** 20/09/2026 – 27/09/2026 · Region: ap-southeast-1 (Singapore)

### Các công việc đã thực hiện trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | ------------ | --------------- | -------------- |
| CN | - Hoàn thành A3–A8 (tiếp nối phần bắt đầu từ 19/09, xem [tuần 4](../1.4-week4/)): Lambda CRUD, API Gateway, test API, frontend S3, test end-to-end, CloudWatch dashboard, alarm và SNS <br> - Chuẩn bị Phần B: tạo IAM group cấp quyền VPC/EC2/ELB | 20/09/2026 | 20/09/2026 | |
| 2 | - Phần B: tạo VPC 2 AZ (B1), Launch Template và security group (B2) | 21/09/2026 | 21/09/2026 | |
| 3 | - CloudTrail (A9) <br> - Tạo Cognito User Pool <br> - Phần B: Target Group, ALB, Auto Scaling, test cân bằng tải (B3–B5), dọn dẹp | 22/09/2026 | 22/09/2026 | |
| 4 | - Tiếp tục phần xác thực: JWT Authorizer, đăng ký/đăng nhập trên frontend, nút Sửa, lọc dữ liệu theo `userId` <br> - Thử Bedrock, Comprehend (bị chặn) <br> - Đẩy website báo cáo lên GitHub | 23/09/2026 | 23/09/2026 | |
| 5 | - Sửa lỗi 404 GitHub Pages <br> - Thử Translate, Polly; làm tính năng "Đọc ghi chú" bằng Amazon Polly <br> - Mở case AWS Support xin quyền Bedrock | 24/09/2026 | 24/09/2026 | |

### Chi tiết thực hiện

#### 1. Backend serverless (A3–A5)

**Các bước**
* A0–A2 (IAM user, bảng DynamoDB `Notes`, role `LambdaNotesExecutionRole`) đã làm ngày 19/09, xem [tuần 4](../1.4-week4/).
* **A3 – Lambda:** hàm `notes-api` (Python 3.12) xử lý GET/POST/PUT/DELETE; chạy Test event và kiểm tra item trong DynamoDB.
* **A4 – API Gateway:** HTTP API `notes-http-api`, 4 route `GET /notes`, `POST /notes`, `PUT /notes/{noteId}`, `DELETE /notes/{noteId}`; bật CORS.
* **A5 – Test API:** test đủ 4 method bằng PowerShell (`Invoke-RestMethod`), test preflight bằng request OPTIONS.

**Lỗi và cách xử lý**
* CloudShell báo thiếu quyền → gắn thêm `AWSCloudShellFullAccess`. Sau đó CloudShell báo tài khoản đang được xác minh (tới 2 ngày) → chuyển sang test bằng PowerShell trên máy cá nhân.
* `GET /notes` trả về sai thứ tự do `Scan()` không đảm bảo thứ tự → sắp xếp theo `createdAt` giảm dần trong Lambda.

**Kết quả**
* 4 route CRUD hoạt động, dữ liệu ghi/sửa/xoá đúng trong DynamoDB; PUT giữ nguyên `createdAt`.
* Preflight OPTIONS trả về 204 kèm đủ header CORS.

#### 2. Frontend, giám sát, kiểm toán (A6–A9)

**Các bước**
* **A6 – S3:** bucket `cloudnote-app-0205568-2026`, bật Static website hosting, Bucket Policy cho phép đọc công khai; `index.html` viết bằng HTML/CSS/JS thuần, có hàm thoát ký tự HTML để chống XSS.
* **A7 – Test end-to-end:** thêm/xoá ghi chú trên web, tải lại trang và đối chiếu với DynamoDB.
* **A8 – CloudWatch:** dashboard `CloudNote-Dashboard` (Lambda: Duration, Errors, Invocations; DynamoDB: Consumed Read/Write Capacity), alarm `notes-api-error-alarm` (Lambda Errors > 0 trong 5 phút), SNS topic `cloudnote-alerts` gửi email.
* **A9 – CloudTrail:** trail `cloudnote-audit-trail` (multi-region), ghi Management events Read + Write.
* **Chi phí:** tạo Zero-Spend Budget.

**Lỗi và cách xử lý**
* Alarm ở trạng thái "Insufficient data" do thiếu datapoint → đặt "Treat missing data as good", alarm chuyển OK.
* IAM user không xem được Billing (mặc định AWS chặn) → đăng nhập root một lần để tạo budget rồi đăng xuất.

**Kết quả**
* Thêm/xoá/tải lại trên web đều đúng; ghi chú mới nhất nằm đầu danh sách.
* Dashboard có số liệu; alarm OK; email SNS đã xác nhận; CloudTrail đang Logging.

#### 3. Phần B: VPC, EC2, ALB, Auto Scaling (bài thực hành riêng)

**Các bước**
* Cấp quyền VPC/EC2/ELB qua IAM group `cloudnote-network-group` (user đã đạt giới hạn 10 managed policy gắn trực tiếp).
* **B1:** VPC `cloudnote-vpc` (10.0.0.0/16), 2 public subnet ở 2 AZ, Internet Gateway `cloudnote-igw`, route table `cloudnote-rtb-public`, không dùng NAT Gateway; bật auto-assign public IPv4 cho subnet.
* **B2:** Launch Template `cloudnote-web-template` (Amazon Linux 2023, t3.micro), security group `cloudnote-web-sg` (HTTP 80 mở, SSH 22 chỉ cho IP cá nhân); User Data cài Apache, lấy Instance ID qua IMDSv2.
* **B3:** Target Group `cloudnote-tg`, ALB `cloudnote-alb` (Internet-facing, listener HTTP:80).
* **B4:** Auto Scaling Group `cloudnote-asg` (Desired 2 / Min 1 / Max 2), bật EC2 và ELB health check.
* **B5:** truy cập DNS của ALB, tải lại nhiều lần để kiểm tra cân bằng tải.
* **Dọn dẹp:** xoá theo thứ tự ASG → ALB → Target Group → Launch Template → VPC → Key Pair.

**Lỗi và cách xử lý**
* Wizard VPC mặc định 0 public / 2 private subnet → chỉnh thành 2 public / 0 private.
* ALB bị chọn thêm security group `default` → bỏ, chỉ giữ `cloudnote-web-sg`.
* ASG để "No load balancer" → chọn "Attach to an existing load balancer" và Target Group `cloudnote-tg`.
* Trình duyệt báo time-out khi mở DNS của ALB → kiểm tra bằng `nslookup` và `curl` thấy hạ tầng bình thường; lỗi do cache/tiện ích trình duyệt, mở bằng cửa sổ ẩn danh thì chạy.

**Kết quả**
* 2 instance Healthy; ALB trả HTTP 200; Instance ID luân phiên giữa 2 máy khi tải lại.
* Đã xoá toàn bộ tài nguyên Phần B.

#### 4. Xác thực người dùng và cách ly dữ liệu

**Các bước**
* **Cognito:** User Pool đăng nhập bằng email, cho phép tự đăng ký; App Client kiểu Single-page application (không có client secret).
* **API Gateway:** JWT Authorizer trên `notes-http-api` (Issuer là User Pool, Audience là App Client), gắn vào cả 4 route.
* **Frontend:** form Đăng ký / Xác nhận email / Đăng nhập / Đăng xuất (thư viện `amazon-cognito-identity-js`); hàm `authFetch()` gắn header `Authorization: Bearer <token>` vào mọi request; thêm nút **Sửa** dùng route PUT có sẵn.
* **Lambda `notes-api`:** lấy `sub` từ claims JWT làm `userId`; GET chỉ trả ghi chú của người gọi; POST gắn `userId`; PUT/DELETE dùng `ConditionExpression` kiểm tra chủ sở hữu, không khớp trả 403.

**Lỗi và cách xử lý**
* `AccessDeniedException` khi mở Cognito → gắn `AmazonCognitoPowerUser` cho user.
* Preflight bị chặn khi gửi kèm token do CORS thiếu header `Authorization` → bổ sung `Authorization` vào cấu hình CORS.
* Lỗi 500 khi tạo ghi chú do DynamoDB không nhận kiểu `float` cho `createdAt` → chuyển sang `Decimal`.
* Ghi chú cũ không còn hiển thị sau khi lọc theo `userId` vì chưa có trường này → đúng hành vi, đó là dữ liệu test.

**Kết quả**
* Đăng ký, xác nhận email, đăng nhập, đăng xuất hoạt động.
* Gọi API không có token → **401 Unauthorized**; có token hợp lệ → Thêm/Sửa/Xoá bình thường.
* Thử với 2 tài khoản: mỗi tài khoản chỉ thấy ghi chú của mình.

#### 5. Tính năng AI: "Đọc ghi chú" bằng Amazon Polly

**Các bước**
* Thử Amazon Bedrock và Amazon Comprehend, sau đó Amazon Translate: cả ba bị chặn vì tài khoản đang ở Free account plan. Không nâng cấp lên Paid plan để tránh phát sinh chi phí.
* Thử Amazon Polly trên Console: tiếng Anh đọc tốt (engine Standard, giọng Joanna); tiếng Việt phát âm không chuẩn vì không có giọng vi-VN → chọn Polly, demo bằng ghi chú tiếng Anh.
* **Quyền:** inline policy `PollySynthesizeSpeech` cho `LambdaNotesExecutionRole`, chỉ gồm `polly:SynthesizeSpeech`.
* **API Gateway:** route `POST /notes/{noteId}/speak`, gắn JWT Authorizer, integration Lambda mới.
* **Lambda:** hàm `speak_note` kiểm tra ghi chú thuộc người gọi rồi mới gọi Polly, trả về MP3; phân nhánh theo `routeKey`; tăng timeout từ 3 lên 10 giây.
* **Frontend:** nút "🔊 Đọc" gọi API qua `authFetch` và phát MP3.
* Mở case AWS Support xin quyền Bedrock (Bedrock Allowlisting), đang chờ phản hồi.

**Lỗi và cách xử lý**
* Thiếu quyền `polly:DescribeVoices` và `support:DescribeSupportLevel` trên Console → thêm inline policy `ConsoleTestPolly` và `AWSSupportAccess` vào group `cloudnote-network-group`.
* Không thấy API trong API Gateway do Console đang ở us-east-1 → chuyển về ap-southeast-1.
* Request `/speak` có thể bị nhánh POST tạo ghi chú xử lý nhầm → phân nhánh theo `routeKey`, đặt nhánh `/speak` trước.
* Lần test "ghi chú của người khác" đầu tiên không hợp lệ (gửi nhầm chuỗi giữ chỗ thay vì noteId thật) → lấy noteId thật của tài khoản A, test lại bằng tài khoản B.

**Kết quả**
* Bấm "🔊 Đọc" khi đã đăng nhập: preflight 204, request 200, phát MP3 thành công.
* Gọi `/speak` không có token: **401**.
* Tài khoản B gọi `/speak` trên ghi chú của tài khoản A: **403**, Lambda không gọi Polly.
* noteId không tồn tại: **403**, không tiết lộ ghi chú có tồn tại hay không.

#### 6. Website báo cáo thực tập

* Đổi `git remote` về repository cá nhân, push lên nhánh `main`; workflow GitHub Actions build Hugo và deploy.
* **Lỗi 404 dù workflow thành công:** workflow đẩy site vào nhánh `gh-pages`, trong khi Settings → Pages đang chọn Source là "GitHub Actions" → đổi sang "Deploy from a branch", nhánh `gh-pages`. Website truy cập được.

### Kiến trúc cuối cùng của CloudNote

Trình duyệt tải trang web tĩnh từ Amazon S3. Người dùng đăng nhập bằng Amazon Cognito và nhận JWT. Mọi lời gọi API tới API Gateway đều kèm JWT; JWT Authorizer kiểm tra token trước khi chuyển request tới Lambda `notes-api`. Lambda đọc/ghi bảng DynamoDB `Notes`, luôn lọc theo `userId` của người gọi, và gọi Amazon Polly cho tính năng "Đọc ghi chú". Amazon CloudWatch giám sát và cảnh báo lỗi, AWS CloudTrail ghi lại các lời gọi API. Phần B (VPC, ALB, Auto Scaling) là bài thực hành riêng, đã dọn dẹp, không thuộc kiến trúc CloudNote.

### Chưa thực hiện

* Amazon ECS / Docker: chưa kịp thực hành, ưu tiên hoàn thiện dự án tổng hợp.

### Kết quả đạt được tuần 5:

* CloudNote chạy end-to-end trên S3, API Gateway, Lambda, DynamoDB, có đăng nhập Cognito và cách ly dữ liệu theo người dùng (401/403 đúng như thiết kế).
* Có tính năng "Đọc ghi chú" bằng Amazon Polly với kiểm tra quyền sở hữu.
* Có giám sát (CloudWatch dashboard, alarm, SNS) và kiểm toán (CloudTrail); kiểm soát chi phí bằng Zero-Spend Budget.
* Hoàn thành thực hành VPC/ALB/Auto Scaling và xoá sạch tài nguyên sau khi test.
* Website báo cáo đã chạy trên GitHub Pages.
