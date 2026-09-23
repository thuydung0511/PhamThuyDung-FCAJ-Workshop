---
title: "GitHub OIDC"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Mục tiêu

Xác thực **GitHub Actions** với AWS qua **OIDC** — không cần `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` tĩnh trong repository secrets.

![GitHub OIDC → AWS (không dùng access key tĩnh)](/images/5-Workshop/image9.png)

## Bước 1 — Thêm GitHub làm OIDC provider

1. **IAM → Identity providers → Add provider**.
2. Provider type: **OpenID Connect**.
3. Provider URL: `https://token.actions.githubusercontent.com`
4. Audience: `sts.amazonaws.com`
5. Create provider.

![Thêm GitHub OIDC provider](/images/5-Workshop/image10.png)

## Bước 2 — Tạo IAM role cho GitHub Actions

1. **IAM → Roles → Create role**.
2. Trusted entity: **Web identity** → chọn GitHub OIDC provider.
3. Condition (ví dụ):

```json
"StringEquals": {
  "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
},
"StringLike": {
  "token.actions.githubusercontent.com:sub": "repo:thuydung0511/CloudNote:*"
}
```

> Phạm vi repo là tạm thời — link repository CloudNote sẽ được cập nhật sau.

4. Role name: `GitHubActionsCloudNoteDeploy` (hoặc tên bạn chọn).
5. Gắn permissions policy từ [5.4 IAM](5.4-IAM/).

![Gắn permissions policy cho role](/images/5-Workshop/image11.png)

![Trust policy của role](/images/5-Workshop/image12.png)

## Bước 3 — Cấu hình repository secrets

Trong **GitHub → Settings → Secrets and variables → Actions**, thêm:

| Secret | Mục đích |
|--------|----------|
| `AWS_ROLE_ARN` | ARN của role OIDC |
| `AWS_REGION` | `ap-southeast-1` |
| `APP_BUCKET` | Tên bucket S3 frontend CloudNote |
| `API_BASE_URL` | Base URL API Gateway (`/notes`) |

**Không** lưu access key tĩnh.

![Repository secrets](/images/5-Workshop/image13.png)

![Repository secrets (tiếp)](/images/5-Workshop/image14.png)

## Bước 4 — Cập nhật GitHub Actions workflow

Trong `.github/workflows/deploy.yml`:

```yaml
permissions:
  id-token: write
  contents: read

- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
    aws-region: ap-southeast-1
```

## Bước 5 — Xác minh OIDC login

1. Push commit để chạy workflow.
2. Xác nhận bước **Configure AWS credentials (OIDC)** thành công.
3. Xác nhận pipeline hoàn tất — deploy Lambda + sync frontend S3.

![Pipeline CI/CD thành công qua OIDC](/images/5-Workshop/image18.png)

![Lambda được cập nhật sau deploy](/images/5-Workshop/image19.png)

## Kết quả mong đợi

- CI dùng credential ngắn hạn qua `AssumeRoleWithWebIdentity`
- Không có access key tĩnh trong GitHub secrets
- Cùng một role có thể xoay/giới hạn bằng cập nhật IAM