---
title: "Tổng Quan Workshop"
date: "2025-12-05"
weight: 1
chapter: false
pre: "<b>5.1. </b>"
---

## 5.1 Tổng Quan Workshop  
**Amazon Cognito và AWS Amplify là gì? • Tại sao nên dùng chung?**

### Amazon Cognito là gì?

**Amazon Cognito** là dịch vụ quản lý danh tính (Identity Service) do AWS cung cấp, với các khả năng:

- Đăng ký / đăng nhập người dùng (email/password, mạng xã hội, SAML/OIDC)
- Xác thực email / số điện thoại tích hợp sẵn
- Hỗ trợ đa yếu tố (MFA)
- Thư mục người dùng (User Directory) với thuộc tính tùy chỉnh và nhóm người dùng
- Cấp phát token bảo mật (ID token, Access token, Refresh token)
- Tự động mở rộng đến hàng triệu người dùng
- Free tier: 50.000 MAU (Monthly Active Users)

Trong workshop này, chúng ta sẽ dùng **Cognito User Pools** làm dịch vụ xác thực backend chính.

---

### AWS Amplify (Gen 2 / v6+) là gì?

**AWS Amplify** là bộ công cụ dành cho lập trình viên frontend/mobile khi làm việc với AWS.

Phiên bản **Amplify Gen 2 (v6+)**, ra mắt giai đoạn 2024–2025, đã được viết lại hoàn toàn và có các ưu điểm:

- Tree-shakable (chỉ import những gì cần)
- Tương thích Server-Side Rendering (SSR) đầy đủ với cấu hình `ssr: true`
- Tối ưu cho Next.js (App Router + React Server Components)
- Không cần `@aws-amplify/ui-react` hay Hosted UI nữa
- Sử dụng kiến trúc modular mới: `@aws-amplify/auth`, `@aws-amplify/datastore`, …

Trong workshop, ta chỉ dùng **Amplify Auth** — cách đơn giản, an toàn và hiện đại nhất để kết nối Next.js với Cognito.

---

### Tại sao dùng Cognito + Amplify Gen 2 vào năm 2025?

| Lợi ích                         | Giải thích                                                                                  |
|----------------------------------|----------------------------------------------------------------------------------------------|
| **Phát triển nhanh & an toàn**      | Amplify xử lý token, refreshing, và toàn bộ API của Cognito                                  |
| **Tương thích SSR & App Router**    | Hoạt động hoàn hảo với `ssr: true` → không lỗi hydration, không lỗi `window is not defined` |
| **Không cần Client Secret**         | SPA client an toàn (không dùng secret) → phù hợp cho Next.js & Vercel                       |
| **Bảo mật cấp doanh nghiệp**        | Token không nằm trong localStorage trên server, refresh tự động, mặc định an toàn           |
| **Phân quyền theo vai trò**         | Sử dụng Cognito Groups → có `cognito:groups` trong ID token                                 |
| **Triển khai ở bất cứ đâu**         | Hoạt động tốt trên Vercel, Netlify, Amplify Hosting, CloudFront + S3                        |
| **Tương lai lâu dài**               | Kiến trúc được AWS khuyến nghị chính thức từ 2024 trở đi                                     |

---

### Bạn sẽ xây dựng được gì?

Một ứng dụng Next.js 14 (App Router) hoàn chỉnh với:

- Sign-up → gửi mã xác minh email tự động
- Sign-in → điều hướng vào dashboard bảo vệ
- `AuthContext` toàn cục + hook `useAuth()`
- Component `ProtectedRoute`
- Giao diện hiện đại bằng shadcn/ui
- Không có bất kỳ anti-pattern bảo mật nào

---

### Sẵn sàng chưa?

Bạn đang đi đúng hướng giống như cách mà các Solutions Architect của AWS, startup và doanh nghiệp triển khai trong năm 2025.

---

**Bước tiếp theo:** [5.2 Prerequisites](../5.2-Prerequiste/) → Thiết lập AWS, Node.js và môi trường phát triển
