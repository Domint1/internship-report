---
title: "Implement Auth Context & Protected Routes"
date: "2025-12-05"
weight: 3
chapter: false
pre: "<b>5.5.3 </b>"
---

## 5.5.3 Implement Auth Context & Protected Routes  
**Global authentication state + automatic route protection**

You’ve built the Cognito logic and the beautiful UI.  
Now you’ll connect everything with a **global `AuthContext`** and a **smart `ProtectedRoute`** — exactly as you designed.

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
        <div className="text-xl font-medium">Loading...</div>
      </div>
    );
  }

  if (!user) {
    return null; // Redirecting...
  }

  return <>{children}</>;
}
```

Congratulations! You've built a 2025-standard, AWS-native authentication flow that top-tier teams use daily.

---

**Navigation:**
- **Previous:** [5.5.2 Build Authentication UI](../5.5.2-auth-ui/)
- **Next Step:** [5.5.4 Project Demo](../5.5.4-use-project-demo/) → Run and test the complete authentication system