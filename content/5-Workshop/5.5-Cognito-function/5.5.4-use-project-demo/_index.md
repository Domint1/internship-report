---
title: "Use demo project for cognito authentication"
date: "2025-12-05"
weight: 3
chapter: false
pre: "<b>5.5.4 </b>"
---

## 5.5.4 Use demo project for cognito authentication
**Global authentication state + automatic route protection**

I have sourecode contain the UI . So You’ve built the Cognito logic  
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

when done all thing . i have to add protected to dashboard to block people dont login to access dashboard and protected page .

just add protected page like this 

```tsx
"use client"

import { ProtectedRoute } from "@/components/ProtectedRoute"
import { useAuth } from "@/context/AuthContext"
import { LogOut, User, Mail, Shield, KeyRound, ArrowRight } from "lucide-react"
import Link from "next/link"

export default function DashboardPage() {
  const { user, signOut } = useAuth()

  const handleSignOut = async () => {
    await signOut()
  }

  return (
    <ProtectedRoute>
      <div className="min-h-screen bg-gradient-to-br from-background via-background to-secondary/5">
        {/* Header */}
        <header className="border-b border-border/50 backdrop-blur-sm sticky top-0 z-50">
          <div className="max-w-6xl mx-auto px-6 py-4 flex items-center justify-between">
            <div className="flex items-center gap-3">
              <div className="w-10 h-10 rounded-lg bg-gradient-to-br from-primary to-primary/60 flex items-center justify-center">
                <User className="w-6 h-6 text-primary-foreground" />
              </div>
              <h1 className="text-xl font-bold text-foreground">Dashboard</h1>
            </div>
            <button
              onClick={handleSignOut}
              className="inline-flex items-center gap-2 px-4 py-2 rounded-lg text-sm font-medium bg-destructive/10 text-destructive hover:bg-destructive/20 transition-colors"
            >
              <LogOut className="w-4 h-4" />
              Sign Out
            </button>
          </div>
        </header>

        {/* Main Content */}
        <main className="max-w-6xl mx-auto px-6 py-12">
          {/* Welcome Section */}
          <div className="mb-12">
            <div className="space-y-2 mb-8">
              <h2 className="text-4xl font-bold text-foreground">Welcome back</h2>
              <p className="text-lg text-muted-foreground">{user?.name || user?.email}</p>
            </div>

            {/* Stats Cards */}
            <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
              {/* Profile Status Card */}
              <div className="group relative overflow-hidden rounded-2xl bg-card border border-border/50 p-6 hover:shadow-lg transition-shadow duration-300">
                <div className="absolute inset-0 bg-gradient-to-br from-primary/5 to-transparent opacity-0 group-hover:opacity-100 transition-opacity" />
                <div className="relative space-y-4">
                  <div className="w-12 h-12 rounded-xl bg-primary/10 flex items-center justify-center">
                    <Shield className="w-6 h-6 text-primary" />
                  </div>
                  <div>
                    <h3 className="text-sm font-semibold text-muted-foreground uppercase tracking-wide">
                      Account Status
                    </h3>
                    <p className="text-2xl font-bold text-foreground mt-2">Active</p>
                  </div>
                </div>
              </div>

              {/* Role Card */}
              <div className="group relative overflow-hidden rounded-2xl bg-card border border-border/50 p-6 hover:shadow-lg transition-shadow duration-300">
                <div className="absolute inset-0 bg-gradient-to-br from-accent/5 to-transparent opacity-0 group-hover:opacity-100 transition-opacity" />
                <div className="relative space-y-4">
                  <div className="w-12 h-12 rounded-xl bg-accent/10 flex items-center justify-center">
                    <User className="w-6 h-6 text-accent" />
                  </div>
                  <div>
                    <h3 className="text-sm font-semibold text-muted-foreground uppercase tracking-wide">User Role</h3>
                    <p className="text-2xl font-bold text-foreground mt-2 capitalize">{user?.role || "Member"}</p>
                  </div>
                </div>
              </div>

              {/* Verification Status */}
              <div className="group relative overflow-hidden rounded-2xl bg-card border border-border/50 p-6 hover:shadow-lg transition-shadow duration-300">
                <div className="absolute inset-0 bg-gradient-to-br from-chart-1/5 to-transparent opacity-0 group-hover:opacity-100 transition-opacity" />
                <div className="relative space-y-4">
                  <div className="w-12 h-12 rounded-xl bg-chart-1/10 flex items-center justify-center">
                    <Mail className="w-6 h-6 text-chart-1" />
                  </div>
                  <div>
                    <h3 className="text-sm font-semibold text-muted-foreground uppercase tracking-wide">
                      Email Status
                    </h3>
                    <p className="text-2xl font-bold text-foreground mt-2">Verified</p>
                  </div>
                </div>
              </div>
            </div>
          </div>

          {/* User Information Section */}
          <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
            {/* Main User Card */}
            <div className="lg:col-span-2 rounded-2xl bg-card border border-border/50 overflow-hidden hover:shadow-lg transition-shadow duration-300">
              <div className="h-24 bg-gradient-to-r from-primary to-accent/50" />
              <div className="p-8 -mt-12 relative">
                <div className="mb-8">
                  <div className="w-24 h-24 rounded-2xl bg-card border-4 border-background shadow-lg flex items-center justify-center mb-6">
                    <div className="w-full h-full bg-gradient-to-br from-primary/20 to-accent/20 rounded-xl flex items-center justify-center">
                      <User className="w-12 h-12 text-primary" />
                    </div>
                  </div>
                  <div>
                    <h3 className="text-2xl font-bold text-foreground">{user?.name || user?.email}</h3>
                    <p className="text-muted-foreground mt-1">Authenticated User</p>
                  </div>
                </div>

                <div className="space-y-4 pt-8 border-t border-border/50">
                  <div className="flex items-start gap-4">
                    <Mail className="w-5 h-5 text-primary mt-1 flex-shrink-0" />
                    <div className="flex-1 min-w-0">
                      <p className="text-sm text-muted-foreground">Email Address</p>
                      <p className="text-foreground font-medium break-all">{user?.email}</p>
                    </div>
                  </div>

                  <div className="flex items-start gap-4">
                    <Shield className="w-5 h-5 text-accent mt-1 flex-shrink-0" />
                    <div className="flex-1">
                      <p className="text-sm text-muted-foreground">User ID</p>
                      <p className="text-foreground font-medium font-mono text-sm">{user?.userId || "N/A"}</p>
                    </div>
                  </div>

                  <div className="flex items-start gap-4">
                    <User className="w-5 h-5 text-chart-1 mt-1 flex-shrink-0" />
                    <div className="flex-1">
                      <p className="text-sm text-muted-foreground">Account Role</p>
                      <p className="text-foreground font-medium capitalize">{user?.role || "Member"}</p>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            {/* Actions Sidebar */}
            <div className="space-y-4">
              <Link
                href="/forgot-password"
                className="flex items-center justify-between gap-3 p-4 rounded-xl bg-card border border-border/50 hover:bg-secondary/50 hover:border-border transition-all duration-200 group"
              >
                <div className="flex items-center gap-3">
                  <KeyRound className="w-5 h-5 text-primary" />
                  <span className="font-medium text-foreground">Change Password</span>
                </div>
                <ArrowRight className="w-4 h-4 text-muted-foreground group-hover:translate-x-1 transition-transform" />
              </Link>

              <button
                onClick={handleSignOut}
                className="w-full flex items-center justify-between gap-3 p-4 rounded-xl bg-destructive/10 border border-destructive/20 hover:bg-destructive/20 transition-all duration-200 group"
              >
                <div className="flex items-center gap-3">
                  <LogOut className="w-5 h-5 text-destructive" />
                  <span className="font-medium text-destructive">Sign Out</span>
                </div>
                <ArrowRight className="w-4 h-4 text-destructive/50 group-hover:translate-x-1 transition-transform" />
              </button>
            </div>
          </div>
        </main>
      </div>
    </ProtectedRoute>
  )
}
```



Congratulations! You've built a 2025-standard, AWS-native authentication flow that top-tier teams use daily.

---

**Navigation:**
- **Previous:** [5.5.3 Implement Protected Routes](../5.5.3-protected-routes/)
- **Next Step:** [5.6 Testing & Verification](../../5.6-Testing/) → Run and verify the complete authentication system