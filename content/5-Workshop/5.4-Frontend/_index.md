---
title: "Frontend on S3"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Goal

Publish the CloudNote user interface (a single `index.html` in plain HTML/CSS/JS) with **S3 static website hosting**, call the API from [5.3](../5.3-backend/) and run an end-to-end test in the browser.

## Steps

### 1. Create the bucket and enable static website hosting

1. **S3 → Create bucket**, name `cloudnote-app-0205568-2026`, Region `ap-southeast-1`.
2. Turn off **Block all public access** (this is a public website) and confirm the warning.
3. **Properties → Static website hosting → Enable**; both Index document and Error document are `index.html`.

### 2. Bucket policy for public read

**Permissions → Bucket policy**, allowing only `s3:GetObject`:

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

### 3. Write `index.html`

The frontend calls the API through the API Gateway Invoke URL:

```javascript
// Invoke URL của API Gateway (bước A4) + hậu tố /notes
const API_URL = "<API_URL>/notes";
```

Note content is user input, so special characters are escaped before being inserted into HTML to prevent XSS:

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

Every displayed value (`title`, `content`, `noteId`) goes through `esc()` when the note list is rendered.

### 4. Upload and test end-to-end

1. **Objects → Upload** `index.html` to the bucket.
2. Open the **bucket website endpoint** and check:
   - The note list loads (CORS works with a real browser).
   - Add a note, reload the page (F5): the note is still there.
   - Delete a note, reload the page: the note is gone.
   - Compare directly in **DynamoDB → Explore table items**.

## Issues and fixes

No issues in this step. A frontend-related issue appeared after adding sign-in (CORS missing the `Authorization` header), see [5.5](../5.5-authentication/).

## Test results

- The S3 website adds, deletes and reloads notes correctly; data is stored in DynamoDB, not in the browser.
- The newest note is shown first.
