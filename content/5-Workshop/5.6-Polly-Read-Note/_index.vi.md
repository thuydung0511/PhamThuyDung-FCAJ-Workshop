---
title: "Tính năng \"Đọc ghi chú\" bằng Amazon Polly"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Mục tiêu

Thêm một tính năng AI cho CloudNote: bấm nút **🔊 Đọc** để nghe nội dung ghi chú, giọng đọc do **Amazon Polly** tạo ra. Tính năng phải giữ đúng các quy tắc xác thực và quyền sở hữu như các route khác.

## Bối cảnh: chọn dịch vụ AI

Tài khoản đang ở **Free account plan**. Em không nâng cấp lên Paid plan để tránh phát sinh chi phí thật, nên thử lần lượt các dịch vụ:

| Dịch vụ | Kết quả |
|---------|---------|
| Amazon Bedrock (Nova Micro, Claude) qua Playground | Bị chặn: cần thêm phương thức thanh toán |
| Amazon Comprehend (Real-time analysis) | Bị chặn: cần thêm phương thức thanh toán |
| Amazon Translate (Real-time translation) | Bị chặn: chuyển tới trang "Free account plan access limitations" |
| Amazon Polly trên Console | Dùng được: tiếng Anh đọc tốt (engine Standard, giọng Joanna); tiếng Việt đọc được nhưng phát âm không chuẩn vì Polly không có giọng vi-VN |

→ Chọn **Amazon Polly**, demo bằng ghi chú tiếng Anh. Song song, em mở một case **AWS Support** xin quyền Bedrock (Account and billing → Account Activation → Bedrock Allowlisting, severity low) theo hướng dẫn của Lab 000001; case đang chờ phản hồi.

## Các bước

### 1. Quyền

- **Cho IAM user (dùng trên Console):** user đã đủ 10 managed policy, nên thêm vào group `cloudnote-network-group` inline policy `ConsoleTestPolly` (`polly:DescribeVoices`, `polly:SynthesizeSpeech`) và managed policy `AWSSupportAccess`.
- **Cho Lambda (least privilege):** thêm inline policy `PollySynthesizeSpeech` vào `LambdaNotesExecutionRole`, chỉ gồm action `polly:SynthesizeSpeech`.

### 2. Route mới trên API Gateway

1. Trên `notes-http-api`, thêm route `POST /notes/{noteId}/speak`.
2. Gắn JWT Authorizer có sẵn ([5.5](../5.5-authentication/)).
3. Tạo **integration Lambda mới** tới `notes-api` (payload 2.0, bật *Grant API Gateway permission to invoke your Lambda function*).
4. Stage `$default` tự động deploy.

### 3. Lambda: hàm `speak_note`

Hàm kiểm tra ghi chú thuộc đúng người gọi rồi mới gọi Polly, trả về MP3 dạng base64:

```python
def speak_note(note_id, user_id):
    # Đọc ghi chú bằng Amazon Polly, trả về MP3 dạng base64.
    item = table.get_item(Key={"noteId": note_id}).get("Item")
    # Note không tồn tại hoặc không thuộc user -> 403 (giống PUT/DELETE,
    # không tiết lộ note của người khác có tồn tại hay không)
    if not item or item.get("userId") != user_id:
        return response(403, {"error": "forbidden"})

    text = f"{item.get('title', '')}. {item.get('content', '')}".strip()
    if not text:
        return response(400, {"error": "note is empty"})

    result = polly.synthesize_speech(
        Text=text[:3000],        # giới hạn độ dài: đúng giới hạn Polly, kiểm soát chi phí
        OutputFormat="mp3",
        VoiceId="Joanna",
        Engine="standard",
    )
    audio_b64 = base64.b64encode(result["AudioStream"].read()).decode("utf-8")
    return response(200, {"audio": audio_b64, "contentType": "audio/mpeg"})
```

Trong `lambda_handler`, nhánh `/speak` nhận diện theo `routeKey` và đặt **trước** nhánh `POST` tạo ghi chú:

```python
        # Route đọc ghi chú (Polly). PHẢI đặt trước nhánh "POST" bên dưới,
        # nếu không request này sẽ bị xử lý nhầm thành "tạo note mới".
        if event.get("routeKey") == "POST /notes/{noteId}/speak" and note_id:
            return speak_note(note_id, user_id)
```

Tăng **timeout** của `notes-api` từ 3 lên 10 giây.

### 4. Frontend: nút "🔊 Đọc"

Mỗi ghi chú có thêm nút "🔊 Đọc" (`noteId` được thoát qua `esc()`). Hàm `speakNote` gọi API qua `authFetch`, xử lý mã lỗi, rồi phát MP3; thông báo trạng thái dùng `textContent` nên không chèn HTML:

```javascript
async function speakNote(noteId, btn) {
  const card = btn.closest(".card");
  const status = card.querySelector(".speak-status");
  const holder = card.querySelector(".speak-audio");
  btn.disabled = true;
  stopAudio();
  status.textContent = "Đang tạo giọng đọc..."; // textContent: không chèn HTML -> an toàn XSS
  try {
    const res = await authFetch(
      `${API_URL}/${encodeURIComponent(noteId)}/speak`,
      { method: "POST" },
    );
    if (res.status === 401) {
      status.textContent =
        "Phiên đăng nhập đã hết hạn. Hãy đăng xuất và đăng nhập lại.";
      return;
    }
    if (res.status === 403) {
      status.textContent = "Bạn không có quyền đọc ghi chú này.";
      return;
    }
    if (!res.ok) {
      status.textContent = `Không đọc được ghi chú (lỗi ${res.status}).`;
      return;
    }

    const data = await res.json();
    // Giải mã MP3 base64 -> Blob -> phát bằng thẻ <audio>
    const bytes = Uint8Array.from(atob(data.audio), (c) =>
      c.charCodeAt(0),
    );
    const blob = new Blob([bytes], {
      type: data.contentType || "audio/mpeg",
    });

    const audio = document.createElement("audio");
    audio.controls = true;
    audio.src = URL.createObjectURL(blob);
    audio.onended = () => {
      status.textContent = "Đã đọc xong.";
    };
    holder.appendChild(audio);
    currentAudio = audio;

    status.textContent = "Đang phát...";
    // Nếu trình duyệt chặn tự phát, người dùng bấm nút ▶ trên thanh audio
    audio.play().catch(() => {
      status.textContent = "Bấm ▶ để nghe.";
    });
  } catch (err) {
    status.textContent =
      "Không đọc được ghi chú. Kiểm tra kết nối rồi thử lại.";
    console.error(err);
  } finally {
    btn.disabled = false;
  }
}
```

Upload lại `index.html` lên bucket S3.

## Lỗi gặp phải và cách xử lý

| Lỗi | Cách xử lý |
|-----|------------|
| Bedrock, Comprehend báo `ValidationException` / not authorized | Giới hạn của Free account plan, không sửa được bằng Console → không dùng, mở case Support xin quyền Bedrock |
| Translate chuyển tới trang "Free account plan access limitations" | Giới hạn của gói, ghi nhận, không nâng cấp |
| `not authorized to perform: polly:DescribeVoices` | Thêm inline policy `ConsoleTestPolly` vào group |
| `not authorized to perform: support:DescribeSupportLevel` | Gắn `AWSSupportAccess` vào group (không dùng root) |
| Không thấy API trong API Gateway | Console đang ở us-east-1 → chuyển về ap-southeast-1 |
| Nút Create case mở khung chat thay vì form | Giao diện Support mới; gửi mô tả qua chat thì hệ thống mở form tạo case |
| Nguy cơ request `/speak` bị xử lý nhầm thành "tạo ghi chú mới" (nhánh `if method == "POST"` bắt mọi POST) | Nhận diện theo `routeKey`, đặt nhánh `/speak` lên trước |
| Nguy cơ lỗi 500 nếu dùng lại integration cũ (quyền invoke có thể chỉ gắn theo route cũ) | Tạo integration mới và cấp quyền cho API Gateway |
| Polly đọc tiếng Việt không chuẩn | Giới hạn của dịch vụ (không có giọng vi-VN) → demo bằng tiếng Anh |
| Lần test "ghi chú của người khác" đầu tiên không hợp lệ (gửi nguyên chuỗi giữ chỗ thay vì noteId thật) | Lấy noteId thật của tài khoản A, test lại bằng tài khoản B |

## Kết quả kiểm thử

| Trường hợp | Kết quả |
|------------|---------|
| Bấm "🔊 Đọc" khi đã đăng nhập | Preflight 204, request **200**, phát MP3 thành công |
| Gọi `/speak` không có token | **401**, bị chặn ngay tại API Gateway |
| Tài khoản B gọi `/speak` trên ghi chú của tài khoản A | **403**, Lambda không gọi Polly |
| Gọi `/speak` với noteId không tồn tại | **403**, không tiết lộ ghi chú có tồn tại hay không |
