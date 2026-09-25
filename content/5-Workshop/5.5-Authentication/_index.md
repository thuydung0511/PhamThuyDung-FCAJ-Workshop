---
title: "User authentication with Cognito"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Goal

- Users must sign up, confirm their email and sign in before using CloudNote.
- The API only accepts requests with a valid JWT.
- Each user can only view, edit and delete their own notes.
- Add an **Edit** button to the interface.

## Steps

### 1. Cognito User Pool and App Client

1. Attach `AmazonCognitoPowerUser` to `cloudnote-dev` ([5.2](../5.2-iam/)).
2. Create a **User Pool** with **Email** sign-in and self-registration enabled.
3. Create an **App Client** of type **Single-page application** (public client, no client secret).

![The User Pool with 3 confirmed users](/images/5-Workshop/5.5-cognito.png)

*Figure: The User Pool with 3 confirmed users. The User name is the sub claim, the same value as the userId column in DynamoDB.*

### 2. JWT Authorizer on API Gateway

1. Open `notes-http-api` → **Authorization** → create a **JWT Authorizer**:
   - Issuer: the Cognito User Pool URL — example format: `https://cognito-idp.<REGION>.amazonaws.com/<USER_POOL_ID>`.
   - Audience: `<APP_CLIENT_ID>`.
2. Attach the authorizer to all 4 routes; the `$default` stage auto-deploys.
3. **CORS**: add `Authorization` to the allowed headers.

![All 5 routes have the JWT Auth authorizer attached](/images/5-Workshop/5.5-authorizer.png)

*Figure: All 5 routes have the JWT Auth authorizer attached.*

### 3. Lambda: read `userId` and isolate data

API Gateway attaches the claims of the validated JWT to the `event`. Lambda uses `sub` (the user's permanent ID) as `userId`:

```python
def get_user_id(event):
    # Lấy "sub" (ID duy nhất, không đổi của người dùng) từ token JWT
    # mà Cognito Authorizer đã xác thực và đính kèm sẵn vào event.
    try:
        return event["requestContext"]["authorizer"]["jwt"]["claims"]["sub"]
    except (KeyError, TypeError):
        return None
```

In `lambda_handler`:

- No `userId` → return `401`:

```python
        # Mọi route khác OPTIONS đều bắt buộc có user_id hợp lệ
        # (Cognito Authorizer đã chặn request không có token hợp lệ từ trước,
        # nhưng kiểm tra lại ở đây cho chắc chắn).
        if user_id is None:
            return response(401, {"error": "unauthorized"})
```

- **GET** returns only the caller's notes:

```python
            items = table.scan(FilterExpression=Attr("userId").eq(user_id)).get("Items", [])
```

- **POST** stores `userId` on the new note; `createdAt` is wrapped in `Decimal`:

```python
            item = {
                "noteId": str(uuid.uuid4()),
                "userId": user_id,
                "title": title,
                "content": content,
                "createdAt": Decimal(str(__import__("time").time())),
            }
```

- **PUT/DELETE** use a `ConditionExpression` so only the owner can edit or delete; a mismatch returns `403`:

```python
            try:
                table.update_item(
                    Key={"noteId": note_id},
                    UpdateExpression="SET title = :t, content = :c",
                    ConditionExpression="userId = :uid",
                    ExpressionAttributeValues={
                        ":t": body.get("title", ""),
                        ":c": body.get("content", ""),
                        ":uid": user_id,
                    },
                )
            except dynamodb.meta.client.exceptions.ConditionalCheckFailedException:
                return response(403, {"error": "forbidden"})
```

`DELETE` works the same way with `ConditionExpression="userId = :uid"`. The CORS headers returned by Lambda also include `Authorization` (`"Access-Control-Allow-Headers": "Content-Type,Authorization"`).

### 4. Frontend: sign-up, sign-in, sending the token

1. Load `amazon-cognito-identity-js` from a CDN and configure the User Pool:

```javascript
const poolData = {
  UserPoolId: "<USER_POOL_ID>",
  ClientId: "<APP_CLIENT_ID>",
};
const userPool = new AmazonCognitoIdentity.CognitoUserPool(poolData);

// Token JWT hiện tại, chỉ giữ trong bộ nhớ JS (KHÔNG lưu localStorage để giảm rủi ro XSS)
let idToken = null;
```

2. Add **Sign up / Confirm email / Sign in / Sign out** forms; the sign-up form has a "Confirm password" field that is checked before calling Cognito.
3. Show the signed-in user's email by reading the JWT payload (for display only, not for authentication).
4. An `authFetch()` function adds the `Authorization` header to every request:

```javascript
// Gọi fetch nhưng tự động gắn header Authorization: Bearer <idToken>
// Dùng chung cho mọi lệnh gọi API (GET/POST/PUT/DELETE) để không lặp code.
function authFetch(url, options = {}) {
  const headers = Object.assign({}, options.headers, {
    Authorization: `Bearer ${idToken}`,
  });
  return fetch(url, Object.assign({}, options, { headers }));
}
```

5. The **Edit** button reuses the "Add note" form: while editing, it calls `PUT` (the route already existed from [5.3](../5.3-backend/)):

```javascript
function startEdit(noteId) {
  const note = currentNotes.find((n) => n.noteId === noteId);
  if (!note) return;
  editingId = noteId;
  document.getElementById("title").value = note.title;
  document.getElementById("content").value = note.content;
  document.getElementById("save-btn").textContent = "Lưu thay đổi";
  document.getElementById("cancel-edit").style.display = "inline-block";
  document.getElementById("title").focus();
}

async function saveNote() {
  const title = document.getElementById("title").value;
  const content = document.getElementById("content").value;
  if (!title) return alert("Nhập tiêu đề!");

  if (editingId) {
    // Chế độ Sửa: gọi PUT tới /notes/{noteId}, giữ nguyên noteId cũ
    await authFetch(`${API_URL}/${editingId}`, {
      method: "PUT",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ title, content }),
    });
  } else {
    // Chế độ Thêm mới: gọi POST như cũ
    await authFetch(API_URL, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ title, content }),
    });
  }
  cancelEdit(); // xoá trắng form + đưa nút về "Thêm ghi chú"
  loadNotes();
}
```

## Issues and fixes

| Issue | Fix |
|-------|-----|
| `AccessDeniedException` when opening Cognito | Attached `AmazonCognitoPowerUser` to the user |
| The preflight was blocked when sending the token | `Access-Control-Allow-Headers` did not include `Authorization` → added it to the API Gateway CORS configuration and to the headers returned by Lambda |
| `500 Internal Server Error` when creating a note | DynamoDB does not accept `float` for `createdAt` → wrapped it in `Decimal(str(...))` before writing |
| Old notes disappeared after filtering by `userId` | Notes created earlier had no `userId` field → expected behavior; they were only test data |

## Test results

- Sign up, email confirmation, sign in and sign out work correctly.
- Calling the API without a token → **401 Unauthorized**.
- With a valid token → Add, Edit and Delete work normally.
- Tested with 2 different accounts: each account sees only its own notes.

![Calling the API without a token returns 401 Unauthorized](/images/5-Workshop/5.5-test-401.png)

*Figure: Calling the API without a token returns 401 Unauthorized.*
