---
title: "Workshop"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Workshop AWS Cognito + Amplify SDK

## Tổng quan

**Amazon Cognito** là dịch vụ quản lý xác thực người dùng, ủy quyền và quản lý người dùng được quản lý hoàn toàn, giúp dễ dàng thêm đăng ký, đăng nhập và kiểm soát truy cập vào các ứng dụng web và mobile.

Trong workshop thực hành này, bạn sẽ:

- Tạo và cấu hình **Cognito User Pool** từ đầu
- Tích hợp **AWS Amplify Gen 2 (v6+)** mới nhất với **Next.js 14 (App Router)**
- Triển khai luồng xác thực hoàn chỉnh:  
  Đăng ký → Xác minh email → Đăng nhập → Protected routes → Truy cập dựa trên vai trò
- Tuân theo các thực hành tốt nhất sẵn sàng cho production (hỗ trợ SSR, xử lý token an toàn, kiến trúc sạch)

**Kết quả cuối cùng:** Một ứng dụng Next.js hoạt động đầy đủ, bảo mật với xác thực người dùng — sẵn sàng deploy trên Vercel, Netlify, hoặc AWS Amplify Hosting.

## Nội dung Workshop

1. **[Tổng quan Workshop](5.1-Workshop-overview/)**
   - Amazon Cognito và AWS Amplify là gì?
   - Tại sao sử dụng chúng cùng nhau?

2. **[Các bước chuẩn bị](5.2-Prerequiste/)**
   - Tài khoản AWS, IAM user, AWS CLI
   - Node.js 18+, npm, VS Code

3. **[Thiết lập AWS Cognito](5.3-AWS-Cognito/)**
   - Tạo User Pool với đăng nhập email
   - Cấu hình chính sách mật khẩu, MFA tùy chọn, App Client (không client secret)
   - Tạo Cognito Groups (admin / user)

4. **[Thiết lập dự án Next.js & Cấu hình Amplify](5.4-Next.js-setup/)**
   - Tạo khung Next.js 14 với App Router + TypeScript + Tailwind CSS
   - Cài đặt `aws-amplify@latest`
   - Cấu hình environment variables và chế độ Amplify SSR

5. **[Authentication Functions, UI và Protected Routes](5.5-Cognito-function/)**
   - Helpers Cognito hoàn chỉnh: signUp, confirmSignUp, signIn, signOut, getCurrentUser, v.v.
   - AuthContext toàn cục + custom `useAuth` hook
   - Trang đăng ký, xác minh email, đăng nhập và quên mật khẩu
   - Component ProtectedRoute
   - Trang dashboard hiển thị thông tin người dùng và vai trò

6. **[Kiểm tra & Xác minh đầy đủ](5.6-Testing/)**
   - Danh sách kiểm tra end-to-end
   - Lỗi phổ biến và mẹo khắc phục

7. **[Dọn dẹp tài nguyên](5.7-Cleanup/)**
   - Xóa User Pool và App Client để tránh phí không mong muốn

---

**Thời gian ước tính:** 3–4 giờ  
**Độ khó:** Trung cấp (khuyến nghị có kiến thức cơ bản về React/Next.js)

Hãy bắt đầu! → Click vào 5.1 **[Tổng quan Workshop](5.1-Workshop-overview/)** để bắt đầu.