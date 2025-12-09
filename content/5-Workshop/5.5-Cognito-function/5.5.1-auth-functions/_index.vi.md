---
title: "Tạo Authentication Functions"
date: "2025-12-05"
weight: 1
chapter: false
pre: "<b>5.5.1 </b>"
---

## 5.5.1 Tạo Authentication Functions  
**Logic Cognito hoàn chỉnh, sẵn sàng cho production — chính xác như được sử dụng trong workshop của bạn**

Bây giờ bạn sẽ tạo **trái tim của hệ thống xác thực** — một file được tổ chức tốt chứa tất cả các hoạt động Cognito với xử lý lỗi phù hợp, kiểu TypeScript và phát hiện vai trò.

### Bước 1 – Sửa tên thư mục & Tạo Config (Đã hoàn thành)
Đảm bảo bạn có:
- `lib/amplify-config.ts` (với `ssr: true`)
- Tên thư mục đã được sửa: `lib/` (không phải `libc/`)

### Bước 2 – Tạo Module Authentication đầy đủ  
**File:** `lib/cognito-auth.ts`

```ts
// lib/cognito-auth.ts
import { Amplify } from 'aws-amplify';
import {
  signUp,
  confirmSignUp,
  signIn,
  signOut,
  getCurrentUser,
  fetchAuthSession,
  fetchUserAttributes,
  resetPassword,
  confirmResetPassword,
  type SignUpInput,
} from 'aws-amplify/auth';

// Import này kích hoạt cấu hình Amplify (side effect)
import '@/lib/amplify-config';

// Tùy chọn: kiểm tra an toàn (hữu ích trong quá trình phát triển)
function ensureAmplifyConfigured() {
  const config = Amplify.getConfig();
  if (!config.Auth?.Cognito?.userPoolId) {
    console.error('Amplify not configured properly. Missing userPoolId.');
    throw new Error('Auth UserPool not configured');
  }
}

// ============================================
// ĐĂNG KÝ
// ============================================
export interface SignUpParams {
  email: string;
  password: string;
  name: string;
}

export async function cognitoSignUp({ email, password, name }: SignUpParams) {
  try {
    const { isSignUpComplete, userId, nextStep } = await signUp({
      username: email.trim(),
      password: password.trim(),
      options: {
        userAttributes: {
          email: email.trim(),
          name: name.trim(),
        },
      },
    } as SignUpInput);

    return {
      success: true,
      isSignUpComplete,
      userId,
      nextStep,
      message: 'Đăng ký thành công! Kiểm tra email của bạn để xác minh.',
    };
  } catch (error: any) {
    console.error('Sign up error:', error);
    if (error.name === 'UsernameExistsException') {
      return { success: false, error: 'Tài khoản đã tồn tại.' };
    }
    return { success: false, error: error.message || 'Đăng ký thất bại' };
  }
}

// ============================================
// XÁC NHẬN ĐĂNG KÝ (Xác minh Email)
// ============================================
export async function cognitoConfirmSignUp(email: string, code: string) {
  try {
    await confirmSignUp({
      username: email.trim(),
      confirmationCode: code.trim(),
    });
    return { success: true, message: 'Email đã được xác minh! Bây giờ bạn có thể đăng nhập.' };
  } catch (error: any) {
    console.error('Confirm sign up error:', error);
    return { success: false, error: error.message || 'Xác minh thất bại' };
  }
}

// ============================================
// ĐĂNG NHẬP
// ============================================
export interface SignInParams {
  email: string;
  password: string;
}

export async function cognitoSignIn({ email, password }: SignInParams) {
  try {
    const { isSignedIn, nextStep } = await signIn({
      username: email.trim(),
      password: password.trim(),
    });

    if (isSignedIn) {
      const attributes = await fetchUserAttributes();
      return {
        success: true,
        isSignedIn,
        user: attributes,
        message: 'Đăng nhập thành công!',
      };
    }

    return { success: false, nextStep, error: 'Cần thêm các bước bổ sung' };
  } catch (error: any) {
    console.error('Sign in error:', error);
    if (error.name === 'NotAuthorizedException') {
      return { success: false, error: 'Email hoặc mật khẩu không hợp lệ' };
    }
    if (error.name === 'UserNotConfirmedException') {
      return { success: false, error: 'Vui lòng xác minh email của bạn trước' };
    }
    return { success: false, error: error.message || 'Đăng nhập thất bại' };
  }
}

// ============================================
// ĐĂNG XUẤT
// ============================================
export async function cognitoSignOut() {
  try {
    await signOut();
    return { success: true, message: 'Đăng xuất thành công' };
  } catch (error: any) {
    console.error('Sign out error:', error);
    return { success: false, error: error.message || 'Đăng xuất thất bại' };
  }
}

// ============================================
// LẤY NGƯỜI DÙNG HIỆN TẠI + PHÁT HIỆN VAI TRÒ
// ============================================
export async function getCognitoUser() {
  try {
    // Điều này sẽ throw nếu không được xác thực
    const currentUser = await getCurrentUser();
    const attributes = await fetchUserAttributes();
    const session = await fetchAuthSession();

    const idTokenPayload = session.tokens?.idToken?.payload;
    const groups = (idTokenPayload?.['cognito:groups'] as string[]) || [];
    const role = groups.includes('admin') ? 'admin' : 'user';

    return {
      userId: currentUser.userId,
      username: currentUser.username,
      email: attributes.email || '',
      name: attributes.name || '',
      groups,
      role,
    };
  } catch (error: any) {
    // Dự kiến khi người dùng không đăng nhập
    if (error.name === 'UserUnAuthenticatedException' || error.message?.includes('not authenticated')) {
      return null;
    }
    console.error('Get user error:', error);
    return null;
  }
}

// ============================================
// ĐẶT LẠI MẬT KHẨU (Bonus)
// ============================================
export async function cognitoResetPassword(email: string) {
  try {
    await resetPassword({ username: email.trim() });
    return { success: true, message: 'Mã đặt lại đã được gửi đến email của bạn' };
  } catch (error: any) {
    return { success: false, error: error.message || 'Không thể gửi mã đặt lại' };
  }
}

export async function cognitoConfirmResetPassword(email: string, code: string, newPassword: string) {
  try {
    await confirmResetPassword({
      username: email.trim(),
      confirmationCode: code.trim(),
      newPassword,
    });
    return { success: true, message: 'Đặt lại mật khẩu thành công!' };
  } catch (error: any) {
    return { success: false, error: error.message || 'Không thể đặt lại mật khẩu' };
  }
}
```


---

**Điều hướng:**
- **Trước:** [5.5 Tổng quan Cognito Functions](../)
- **Bước tiếp theo:** [5.5.2 Xây dựng Authentication UI](../5.5.2-auth-ui/) → Tạo các trang đăng ký, đăng nhập và dashboard
