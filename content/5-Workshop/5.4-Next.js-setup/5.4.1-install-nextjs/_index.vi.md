---
title : "Cài đặt dự án Next.js"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

### Chuẩn bị môi trường

Trong bước này, bạn sẽ tạo một dự án Next.js mới và cài đặt các thư viện UI cần thiết sẽ được sử dụng sau này trong giao diện xác thực.

---

### 1. Tạo dự án Next.js mới

Nếu bạn bắt đầu từ đầu, chạy lệnh sau:

```bash
npx create-next-app@latest my-cognito-app
```
Khi được hỏi:
Ok to proceed? (y)

Chọn y, sau đó di chuyển vào thư mục dự án:
```bash
cd my-cognito-app
```

### 2. Cài đặt shadcn/ui (Thư viện UI Component)

Bạn sẽ sử dụng shadcn/ui để xây dựng giao diện xác thực sau này.

Khởi tạo shadcn:
```bash
npx shadcn@latest init
```

Điều này sẽ thiết lập trình tạo component cho dự án của bạn.

Môi trường của bạn giờ đây đã sẵn sàng cho các bước tiếp theo, nơi bạn sẽ cài đặt Amplify SDK và cấu hình tích hợp AWS Cognito.

---

**Điều hướng:**
- **Trước:** [5.4 Thiết lập dự án Next.js](../) 
- **Bước tiếp theo:** [5.4.2 Cài đặt Amplify SDK](../5.4.2-install-amplify/) → Cài đặt và cấu hình Amplify SDK




