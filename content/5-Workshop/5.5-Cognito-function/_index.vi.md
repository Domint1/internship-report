---
title : "Authentication Functions, UI và Protected Routes"
date : "`r Sys.Date()`"
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

### Authentication Functions, UI và Protected Routes

Trong phần này, bạn sẽ tích hợp **AWS Cognito** vào ứng dụng Next.js bằng cách sử dụng **Amplify SDK**.  
Bạn sẽ triển khai logic xác thực cốt lõi, xây dựng giao diện thân thiện với người dùng và bảo mật các trang cụ thể bằng protected routes.

Khi kết thúc phần này, ứng dụng của bạn sẽ hỗ trợ:

---

### ✔ Tính năng bạn sẽ triển khai

- **Sign Up Function**  
  Tạo người dùng mới trong Cognito (với xác minh email).

- **Sign In Function**  
  Xác thực người dùng và nhận token do Cognito phát hành.

- **Sign Out Function**  
  Kết thúc phiên đã xác thực một cách đúng đắn thông qua Amplify.

- **Session Handling**  
  Kiểm tra xem người dùng đã đăng nhập chưa và lấy thông tin phiên hiện tại.

- **Protected Routes**  
  Hạn chế quyền truy cập vào một số trang nhất định trừ khi người dùng đã được xác thực.

- **UI Components**  
  Xây dựng các trang xác thực đơn giản như Login, Register, Profile và Dashboard bằng thư viện UI bạn chọn.

> **Tùy chọn nhanh**
>
> Bạn có thể sử dụng live demo để thiết lập nhanh hoặc làm theo hướng dẫn từng bước bên dưới để chỉnh sửa functions và UI trực tiếp.
>
> > Bạn có thể sử dụng live demo để thiết lập nhanh hoặc làm theo hướng dẫn từng bước bên dưới:
>
> 1. [Tạo Cognito Authentication Functions](5.5.1-auth-functions/)
> 2. [Xây dựng Authentication UI](5.5.2-auth-ui/)
> 3. [Triển khai Auth Context & Protected Routes](5.5.3-protected-routes/)
> 4. [Sử dụng Project Demo (Quick Start)](5.5.4-use-project-demo/)
>
> **Quick Start Demo:** <https://github.com/Thormastran/my-cognito-project/>

---

**Điều hướng:**
- **Trước:** [5.4 Thiết lập dự án Next.js](../5.4-Next.js-setup/) 
- **Bước tiếp theo:** [5.6 Full Testing & Verification](../5.6-Testing/) → Kiểm tra tất cả tính năng xác thực
làm theo hướng dẫn để tùy chỉnh và giải thích chi tiết.

### Nội dung

1. [Tạo Cognito Authentication Functions](5.5.1-auth-functions/)
2. [Xây dựng Authentication UI](5.5.2-auth-ui/)
3. **[Triển khai Auth Context & Protected Routes](5.5.3-protected-routes/)**
4.**[Full Testing & Verification](5.5.4-use-project-demo)**
