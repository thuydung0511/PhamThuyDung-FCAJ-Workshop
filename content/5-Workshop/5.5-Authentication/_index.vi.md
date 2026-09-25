---
title: "Xác thực người dùng với Cognito"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Mục tiêu

- Người dùng phải đăng ký, xác nhận email và đăng nhập mới dùng được CloudNote.
- API chỉ nhận request có JWT hợp lệ.
- Mỗi người chỉ xem, sửa, xoá được ghi chú của chính mình.
- Bổ sung nút **Sửa** ghi chú trên giao diện.

## Các bước

### 1. Cognito User Pool và App Client

1. Gắn `AmazonCognitoPowerUser` cho `cloudnote-dev` ([5.2](../5.2-iam/)).
2. Tạo **User Pool** đăng nhập bằng **Email**, cho phép người dùng tự đăng ký (self-registration).
3. Tạo **App Client** kiểu **Single-page application** (public client, không có client secret).

![User Pool với 3 user đã xác nhận](/images/5-Workshop/5.5-cognito.png)

*Hình: User Pool với 3 user đã xác nhận. User name chính là claim sub, trùng với cột userId trong DynamoDB.*

### 2. JWT Authorizer trên API Gateway

1. Mở `notes-http-api` → **Authorization** → tạo **JWT Authorizer**:
   - Issuer: URL của Cognito User Pool — ví dụ định dạng: `https://cognito-idp.<REGION>.amazonaws.com/<USER_POOL_ID>`.
   - Audience: `<APP_CLIENT_ID>`.
2. Gắn authorizer vào cả 4 route; stage `$default` tự động deploy.
3. **CORS**: thêm `Authorization` vào danh sách header được phép.

![Cả 5 route đều gắn JWT Auth](/images/5-Workshop/5.5-authorizer.png)

*Hình: Cả 5 route đều gắn JWT Auth.*

### 3. Lambda: lấy `userId` và cách ly dữ liệu

API Gateway đính kèm các claim của JWT đã xác thực vào `event`. Lambda lấy `sub` (ID cố định của người dùng) làm `userId`:

```python
def get_user_id(event):
    # Lấy "sub" (ID duy nhất, không đổi của người dùng) từ token JWT
    # mà Cognito Authorizer đã xác thực và đính kèm sẵn vào event.
    try:
        return event["requestContext"]["authorizer"]["jwt"]["claims"]["sub"]
    except (KeyError, TypeError):
        return None
```

Trong `lambda_handler`:

- Không có `userId` → trả `401`:

```python
        # Mọi route khác OPTIONS đều bắt buộc có user_id hợp lệ
        # (Cognito Authorizer đã chặn request không có token hợp lệ từ trước,
        # nhưng kiểm tra lại ở đây cho chắc chắn).
        if user_id is None:
            return response(401, {"error": "unauthorized"})
```

- **GET** chỉ trả ghi chú của người gọi:

```python
            items = table.scan(FilterExpression=Attr("userId").eq(user_id)).get("Items", [])
```

- **POST** gắn `userId` vào ghi chú mới; `createdAt` được bọc bằng `Decimal`:

```python
            item = {
                "noteId": str(uuid.uuid4()),
                "userId": user_id,
                "title": title,
                "content": content,
                "createdAt": Decimal(str(__import__("time").time())),
            }
```

- **PUT/DELETE** dùng `ConditionExpression` để chỉ chủ sở hữu mới sửa/xoá được; không khớp → `403`:

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

`DELETE` dùng cùng cách với `ConditionExpression="userId = :uid"`. Header CORS trả về từ Lambda cũng có `Authorization` (`"Access-Control-Allow-Headers": "Content-Type,Authorization"`).

### 4. Frontend: đăng ký, đăng nhập, gửi token

1. Nhúng thư viện `amazon-cognito-identity-js` qua CDN và cấu hình User Pool:

```javascript
const poolData = {
  UserPoolId: "<USER_POOL_ID>",
  ClientId: "<APP_CLIENT_ID>",
};
const userPool = new AmazonCognitoIdentity.CognitoUserPool(poolData);

// Token JWT hiện tại, chỉ giữ trong bộ nhớ JS (KHÔNG lưu localStorage để giảm rủi ro XSS)
let idToken = null;
```

2. Thêm form **Đăng ký / Xác nhận email / Đăng nhập / Đăng xuất**; form đăng ký có ô "Nhập lại mật khẩu", kiểm tra khớp trước khi gọi Cognito.
3. Hiển thị email của người đang đăng nhập bằng cách đọc payload của JWT (chỉ để hiển thị, không dùng để xác thực).
4. Hàm `authFetch()` gắn header `Authorization` vào mọi request:

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

5. Nút **Sửa** dùng chung form với "Thêm ghi chú": khi đang sửa thì gọi `PUT` (route đã có từ [5.3](../5.3-backend/)):

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

## Lỗi gặp phải và cách xử lý

| Lỗi | Cách xử lý |
|-----|------------|
| `AccessDeniedException` khi mở Cognito | Gắn `AmazonCognitoPowerUser` cho user |
| Preflight bị chặn khi gửi kèm token | `Access-Control-Allow-Headers` thiếu `Authorization` → thêm vào cấu hình CORS của API Gateway và header trả về trong Lambda |
| `500 Internal Server Error` khi tạo ghi chú | DynamoDB không nhận kiểu `float` cho `createdAt` → bọc bằng `Decimal(str(...))` trước khi ghi |
| Ghi chú cũ không còn hiển thị sau khi lọc theo `userId` | Ghi chú tạo trước đó không có trường `userId` → đúng hành vi mong muốn, đó chỉ là dữ liệu test |

## Kết quả kiểm thử

- Đăng ký, xác nhận email, đăng nhập, đăng xuất hoạt động đúng.
- Gọi API không có token → **401 Unauthorized**.
- Có token hợp lệ → Thêm, Sửa, Xoá hoạt động bình thường.
- Thử với 2 tài khoản khác nhau: mỗi tài khoản chỉ thấy ghi chú của mình.

![Gọi API không có token → 401 Unauthorized](/images/5-Workshop/5.5-test-401.png)

*Hình: Gọi API không có token → 401 Unauthorized.*
