---
title: "Frontend trên S3"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Mục tiêu

Đưa giao diện CloudNote (một file `index.html` viết bằng HTML/CSS/JS thuần) lên **S3 static website hosting**, gọi tới API ở [5.3](../5.3-backend/) và kiểm thử end-to-end qua trình duyệt.

## Các bước

### 1. Tạo bucket và bật static website hosting

1. **S3 → Create bucket**, tên `cloudnote-app-0205568-2026`, Region `ap-southeast-1`.
2. Tắt **Block all public access** (đây là website công khai) và xác nhận cảnh báo.
3. **Properties → Static website hosting → Enable**; Index document và Error document đều là `index.html`.

### 2. Bucket policy cho phép đọc công khai

**Permissions → Bucket policy**, chỉ cho phép `s3:GetObject`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cloudnote-app-0205568-2026/*"
    }
  ]
}
```

### 3. Viết `index.html`

Frontend gọi API qua Invoke URL của API Gateway:

```javascript
// Invoke URL của API Gateway (bước A4) + hậu tố /notes
const API_URL = "<API_URL>/notes";
```

Nội dung ghi chú do người dùng nhập, nên trước khi chèn vào HTML phải thoát ký tự đặc biệt để chống XSS:

```javascript
// Thoát ký tự HTML để nội dung ghi chú không bị hiểu là mã HTML/JS
function esc(s) {
  return String(s ?? "").replace(
    /[&<>"']/g,
    (c) =>
      ({
        "&": "&amp;",
        "<": "&lt;",
        ">": "&gt;",
        '"': "&quot;",
        "'": "&#39;",
      })[c],
  );
}
```

Mọi giá trị hiển thị (`title`, `content`, `noteId`) đều đi qua `esc()` khi dựng danh sách ghi chú.

### 4. Upload và kiểm thử end-to-end

1. **Objects → Upload** file `index.html` lên bucket.
2. Mở **bucket website endpoint** và kiểm tra:
   - Danh sách ghi chú tải được (CORS hoạt động với trình duyệt thật).
   - Thêm ghi chú, tải lại trang (F5): ghi chú vẫn còn.
   - Xoá ghi chú, tải lại trang: ghi chú đã mất.
   - Đối chiếu trực tiếp trong **DynamoDB → Explore table items**.

## Lỗi gặp phải và cách xử lý

Bước này không gặp lỗi. Lỗi liên quan tới frontend xuất hiện sau khi thêm đăng nhập (CORS thiếu header `Authorization`), xem [5.5](../5.5-authentication/).

## Kết quả kiểm thử

- Website trên S3 thêm, xoá và tải lại ghi chú đúng; dữ liệu lưu trên DynamoDB, không lưu trong trình duyệt.
- Ghi chú mới nhất nằm đầu danh sách.
