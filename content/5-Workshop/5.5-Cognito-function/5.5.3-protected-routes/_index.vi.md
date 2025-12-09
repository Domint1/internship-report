---
title: "Triển khai Auth Context & Protected Routes"
date: "2025-12-05"
weight: 3
chapter: false
pre: "<b>5.5.3 </b>"
---

## 5.5.3 Triển khai Auth Context & Protected Routes  
**Trạng thái xác thực toàn cục + bảo vệ route tự động**

Bạn đã xây dựng logic Cognito và giao diện đẹp.  
Bây giờ bạn sẽ kết nối mọi thứ với một **`AuthContext` toàn cục** và một **`ProtectedRoute` thông minh** — chính xác như bạn đã thiết kế.

### 1. Global Auth Context – `context/AuthContext.tsx`

```tsx
'use client';

import { createContext, useContext, useEffect, useState } from 'react';
import { getCognitoUser, cognitoSignOut } from '@/lib/cognito-auth';

interface User {
  userId: string;
  email: string;
  name: string;
  role: 'admin' | 'user';
}

interface AuthContextType {
  user: User | null;
  loading: boolean;
  signOut: () => Promise<void>;
  refreshUser: () => Promise<void>;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  const loadUser = async () => {
    try {
      const userData = await getCognitoUser();
      if (userData) {
        setUser({
          userId: userData.userId,
          email: userData.email,
          name: userData.name,
          role: userData.role === 'admin' ? 'admin' : 'user',
        });
      } else {
        setUser(null);
      }
    } catch (error) {
      console.error('Failed to load user:', error);
      setUser(null);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    loadUser();
  }, []);

  const signOut = async () => {
    try {
      await cognitoSignOut();
      setUser(null);
    } catch (error) {
      console.error('Sign out error:', error);
    }
  };

  return (
    <AuthContext.Provider value={{ user, loading, signOut, refreshUser: loadUser }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```

### 2. Protected Route Component – `components/ProtectedRoute.tsx`
```tsx
'use client';

import { useEffect } from 'react';
import { useRouter } from 'next/navigation';
import { useAuth } from '@/context/AuthContext';

export function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { user, loading } = useAuth();
  const router = useRouter();

  useEffect(() => {
    if (!loading && !user) {
      router.push('/signin');
    }
  }, [user, loading, router]);

  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="text-xl font-medium">Đang tải...</div>
      </div>
    );
  }

  if (!user) {
    return null; // Đang chuyển hướng...
  }

  return <>{children}</>;
}
```

Chúc mừng! Bạn đã xây dựng một luồng xác thực chuẩn AWS native 2025 mà các đội ngũ hàng đầu sử dụng hàng ngày.

---

**Điều hướng:**
- **Trước:** [5.5.2 Xây dựng Authentication UI](../5.5.2-auth-ui/)
- **Bước tiếp theo:** [5.5.4 Project Demo](../5.5.4-use-project-demo/) → Chạy và kiểm tra hệ thống xác thực hoàn chỉnh
