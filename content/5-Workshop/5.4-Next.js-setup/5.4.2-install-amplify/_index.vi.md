---
title : "Cài đặt Amplify SDK và Cấu hình Amplify"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

### Cài đặt Amplify SDK

Sau khi chuẩn bị dự án Next.js, cài đặt các package Amplify cần thiết:

```bash
npm install aws-amplify @aws-amplify/ui-react
```
### 2. Tạo file cấu hình Amplify

Tạo file mới:

src/lib/amplify-config.ts


Và thêm code sau:

```tsx
import { Amplify } from 'aws-amplify';




export const amplifyConfig = {
    Auth: {
        Cognito: {
            userPoolId: process.env.NEXT_PUBLIC_COGNITO_USER_POOL_ID!,
            userPoolClientId: process.env.NEXT_PUBLIC_COGNITO_APP_CLIENT_ID!,
            loginWith: {
                username: false,
                email: true,
            },
        },
    },
};


// Cấu hình Amplify với hỗ trợ SSR
Amplify.configure(amplifyConfig, {
    ssr: true, // Quan trọng cho Next.js
});
```
### 3. Thêm Environment Variables

Tạo file .env.local và dán Cognito IDs của bạn:

```bash
NEXT_PUBLIC_COGNITO_USER_POOL_ID=us-east-1_xxxxxxxxx
NEXT_PUBLIC_COGNITO_APP_CLIENT_ID=xxxxxxxxxxxxxxxxxxxx
``` 
---

Điều này đảm bảo Amplify có sẵn trong toàn bộ ứng dụng của bạn.



Amplify giờ đây đã được cấu hình và sẵn sàng được sử dụng cho:

Đăng ký

Đăng nhập

Đăng xuất

Quản lý phiên

Protected routes

Tiếp theo, bạn sẽ xây dựng UI cho xác thực.


---

**Điều hướng:**
- **Trước:** [5.4.1 Cài đặt Next.js](../5.4.1-install-nextjs/) 
- **Bước tiếp theo:** [5.5 Cognito Functions](../../5.5-Cognito-function/) → Xây dựng auth functions, UI, và protected routes