# Cài đặt Firebase cho chức năng đồng bộ

App không còn dùng server FastAPI / SQL Server / ngrok. Dữ liệu và tài khoản nằm trên Firebase.

## 1. Tạo dự án
1. Vào https://console.firebase.google.com → **Add project** (gói Spark miễn phí là đủ cho đăng nhập + đồng bộ).
2. **Build → Authentication → Get started → Sign-in method → Email/Password → Enable** (không cần bật "Email link").
3. **Build → Firestore Database → Create database** (chọn *production mode*, chọn khu vực gần bạn, ví dụ `asia-southeast1`).
4. Mở tab **Rules** của Firestore, dán toàn bộ nội dung file `firebase/firestore.rules` rồi **Publish**.

## 2. Điền khóa vào app
Mở `gradle.properties` và điền 2 dòng:
```
firebaseApiKey=<Web API Key>
firebaseProjectId=<Project ID>
```
Lấy ở **Project settings (bánh răng) → General**: *Project ID* và *Web API Key*. Không cần `google-services.json`.
Sau đó **Sync Gradle** và chạy lại app. Nếu để trống, app chạy chế độ ngoại tuyến (chỉ lưu trên máy).

## 3. Cách hoạt động
- Đăng ký: Firebase Auth tạo tài khoản + gửi **email xác thực** (liên kết). Bấm liên kết rồi bấm "Tôi đã xác thực email" trong app.
- Đăng nhập bằng **tên đăng nhập hoặc email** (tên đăng nhập được ánh xạ sang email trong Firestore `usernames/...`).
- Quên mật khẩu: Firebase gửi email đặt lại mật khẩu.
- Đồng bộ: dữ liệu được nén gzip và lưu ở `snapshots/{uid}` (kèm bản liền trước ở `snapshots/{uid}/history/previous`).
  Hai máy cùng sửa → app hỏi giữ bản nào (cơ chế phiên bản giữ nguyên như trước).
- Dữ liệu tối đa khoảng 1 MB sau khi nén (đủ cho hàng chục nghìn giao dịch).

## 4. (Tùy chọn) Email cảnh báo ngân sách
Firebase không tự gửi email tùy ý. App ghi yêu cầu vào collection `mail`; để thật sự gửi, cài extension
**Trigger Email from Firestore** (cần gói Blaze, có hạn mức miễn phí) và cấu hình SMTP (ví dụ Gmail + mật khẩu ứng dụng),
collection = `mail`. Không cài thì thông báo trên máy vẫn hoạt động bình thường.

## 5. Tùy chỉnh email tiếng Việt
**Authentication → Templates**: đổi ngôn ngữ/nội dung email xác thực và đặt lại mật khẩu.
