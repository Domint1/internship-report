---
title: "Xây dựng Authentication UI"
date: "2025-12-05"
weight: 2
chapter: false
pre: "<b>5.5.2 </b>"
---

## 5.5.2 Xây dựng Authentication UI  
**100% trung thành với workshop gốc của bạn — đẹp, chức năng và chuẩn production**

Bây giờ bạn đã có tất cả logic Cognito. Đến lúc xây dựng **giao diện người dùng thực** — chính xác như bạn đã viết trong workshop.

### 1. Trang Landing – `app/page.tsx`

```tsx
// app/page.tsx
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card";

export default function HomePage() {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-gray-50 to-gray-100">
      <Card className="w-full max-w-md">
        <CardHeader className="text-center">
          <CardTitle className="text-3xl font-bold">Chào mừng</CardTitle>
          <CardDescription>Hệ thống quản lý file bảo mật với xác thực người dùng</CardDescription>
        </CardHeader>
        <CardContent className="space-y-4">
          <div className="grid gap-3">
            <Button asChild size="lg">
              <Link href="/signin">Đăng nhập</Link>
            </Button>
            <Button asChild variant="outline" size="lg">
              <Link href="/signup">Đăng ký</Link>
            </Button>
          </div>
          <div className="pt-4 border-t">
            <p className="text-sm text-muted-foreground text-center mb-3">Trang được bảo vệ:</p>
            <div className="grid gap-2">
              <Button asChild variant="secondary" size="sm">
                <Link href="/dashboard">Dashboard</Link>
              </Button>
            </div>
          </div>
        </CardContent>
      </Card>
    </div>
  );
}

```
### 2. Trang Đăng ký (với Xác minh Email) – `app/signup/page.tsx`

```tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { cognitoSignUp, cognitoConfirmSignUp } from '@/lib/cognito-auth';

export default function SignUpPage() {
  const router = useRouter();
  const [step, setStep] = useState<'signup' | 'confirm'>('signup');
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [name, setName] = useState('');
  const [code, setCode] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);

  const handleSignUp = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    setLoading(true);
    const result = await cognitoSignUp({ email, password, name });
    setLoading(false);
    if (result.success) {
      setStep('confirm');
    } else {
      setError(result.error || 'Đăng ký thất bại');
    }
  };

  const handleConfirm = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    setLoading(true);
    const result = await cognitoConfirmSignUp(email, code);
    setLoading(false);
    if (result.success) {
      router.push('/signin');
    } else {
      setError(result.error || 'Xác minh thất bại');
    }
  };

  // Bước xác nhận
  if (step === 'confirm') {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="max-w-md w-full p-8 bg-white rounded-lg shadow-lg">
          <h1 className="text-2xl font-bold mb-6">Xác minh Email</h1>
          <p className="mb-4 text-gray-600">Nhập mã xác minh được gửi đến {email}</p>
          <form onSubmit={handleConfirm}>
            <input
              type="text"
              placeholder="Mã xác minh"
              value={code}
              onChange={(e) => setCode(e.target.value)}
              className="w-full px-4 py-2 border rounded-lg mb-4"
              required
            />
            {error && <p className="text-red-500 mb-4">{error}</p>}
            <button
              type="submit"
              disabled={loading}
              className="w-full bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700 disabled:opacity-50"
            >
              {loading ? 'Đang xác minh...' : 'Xác minh Email'}
            </button>
          </form>
        </div>
      </div>
    );
  }

  // Form đăng ký
  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="max-w-md w-full p-8 bg-white rounded-lg shadow-lg">
        <h1 className="text-2xl font-bold mb-6">Đăng ký</h1>
        <form onSubmit={handleSignUp}>
          <input type="text" placeholder="Họ và tên" value={name} onChange={(e) => setName(e.target.value)} className="w-full px-4 py-2 border rounded-lg mb-4" required />
          <input type="email" placeholder="Email" value={email} onChange={(e) => setEmail(e.target.value)} className="w-full px-4 py-2 border rounded-lg mb-4" required />
          <input type="password" placeholder="Mật khẩu" value={password} onChange={(e) => setPassword(e.target.value)} className="w-full px-4 py-2 border rounded-lg mb-4" required />
          <p className="text-xs text-gray-500 mb-4">
            Mật khẩu phải có ít nhất 8 ký tự với chữ hoa, chữ thường, số và ký tự đặc biệt
          </p>
          {error && <p className="text-red-500 mb-4">{error}</p>}
          <button
            type="submit"
            disabled={loading}
            className="w-full bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700 disabled:opacity-50"
          >
            {loading ? 'Đang đăng ký...' : 'Đăng ký'}
          </button>
        </form>
      </div>
    </div>
  );
}
```

### 3. Trang Đăng nhập – `app/signin/page.tsx`

```tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { cognitoSignIn } from '@/lib/cognito-auth';

export default function LoginPage() {
  const router = useRouter();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    setLoading(true);
    const result = await cognitoSignIn({ email, password });
    setLoading(false);
    if (result.success) {
      router.push('/dashboard');
    } else {
      setError(result.error || 'Đăng nhập thất bại');
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="max-w-md w-full p-8 bg-white rounded-lg shadow-lg">
        <h1 className="text-3xl font-bold mb-6 text-center">Đăng nhập</h1>
        <form onSubmit={handleSubmit}>
          <div className="mb-4">
            <label className="block text-gray-700 mb-2">Email</label>
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500" required />
          </div>
          <div className="mb-6">
            <label className="block text-gray-700 mb-2">Mật khẩu</label>
            <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500" required />
          </div>
          {error && <div className="mb-4 p-3 bg-red-100 border border-red-400 text-red-700 rounded-lg">{error}</div>}
          <button
            type="submit"
            disabled={loading}
            className="w-full bg-blue-600 text-white py-3 rounded-lg font-semibold hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed transition"
          >
            {loading ? 'Đang đăng nhập...' : 'Đăng nhập'}
          </button>
        </form>
        <div className="mt-6 text-center">
          <a href="/signup" className="text-blue-600 hover:underline">Chưa có tài khoản? Đăng ký</a>
        </div>
        <div className="mt-2 text-center">
          <a href="/forgot-password" className="text-gray-600 hover:underline text-sm">Quên mật khẩu?</a>
        </div>
      </div>
    </div>
  );
}
```
### 4. Quên và đổi mật khẩu - ` app/forgot-password/page.tsx`

```tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import Link from 'next/link';
import { cognitoResetPassword, cognitoConfirmResetPassword } from '@/lib/cognito-auth';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Alert, AlertDescription } from '@/components/ui/alert';

export default function ForgotPasswordPage() {
  const [step, setStep] = useState<'request' | 'confirm'>('request');
  const [email, setEmail] = useState('');
  const [code, setCode] = useState('');
  const [newPassword, setNewPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');
  const [error, setError] = useState('');
  const [success, setSuccess] = useState('');
  const [loading, setLoading] = useState(false);
  const router = useRouter();

  const handleRequestReset = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    setSuccess('');
    setLoading(true);

    try {
      const result = await cognitoResetPassword(email);

      if (result.success) {
        setSuccess(result.message || 'Mã đặt lại mật khẩu đã được gửi đến email của bạn');
        setStep('confirm');
      } else {
        setError(result.error || 'Không thể gửi mã đặt lại');
      }
    } catch (err: any) {
      setError(err.message || 'Đã xảy ra lỗi');
    } finally {
      setLoading(false);
    }
  };

  const handleConfirmReset = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    setSuccess('');

    if (newPassword !== confirmPassword) {
      setError('Mật khẩu không khớp');
      return;
    }

    if (newPassword.length < 8) {
      setError('Mật khẩu phải có ít nhất 8 ký tự');
      return;
    }

    setLoading(true);

    try {
      const result = await cognitoConfirmResetPassword(
        email,
        code,
        newPassword,
      );

      if (result.success) {
        setSuccess('Đặt lại mật khẩu thành công! Đang chuyển hướng đến trang đăng nhập...');
        setTimeout(() => {
          router.push('/signin');
        }, 2000);
      } else {
        setError(result.error || 'Không thể đặt lại mật khẩu');
      }
    } catch (err: any) {
      setError(err.message || 'Đã xảy ra lỗi');
    } finally {
      setLoading(false);
    }
  };

  if (step === 'confirm') {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-50 to-indigo-100 px-4">
        <Card className="w-full max-w-md">
          <CardHeader className="space-y-1">
            <CardTitle className="text-2xl font-bold text-center">Đặt lại mật khẩu</CardTitle>
            <CardDescription className="text-center">
              Nhập mã được gửi đến {email} và mật khẩu mới của bạn
            </CardDescription>
          </CardHeader>
          <CardContent>
            <form onSubmit={handleConfirmReset} className="space-y-4">
              {error && (
                <Alert variant="destructive">
                  <AlertDescription>{error}</AlertDescription>
                </Alert>
              )}

              {success && (
                <Alert className="bg-green-50 border-green-200">
                  <AlertDescription className="text-green-800">{success}</AlertDescription>
                </Alert>
              )}

              <div className="space-y-2">
                <Label htmlFor="code">Mã xác minh</Label>
                <Input
                  id="code"
                  type="text"
                  placeholder="Nhập mã 6 chữ số"
                  value={code}
                  onChange={(e) => setCode(e.target.value)}
                  required
                  disabled={loading}
                  maxLength={6}
                />
              </div>

              <div className="space-y-2">
                <Label htmlFor="newPassword">Mật khẩu mới</Label>
                <Input
                  id="newPassword"
                  type="password"
                  placeholder="••••••••"
                  value={newPassword}
                  onChange={(e) => setNewPassword(e.target.value)}
                  required
                  disabled={loading}
                />
                <p className="text-xs text-gray-500">
                  Phải có ít nhất 8 ký tự với chữ hoa, chữ thường, số và ký tự đặc biệt
                </p>
              </div>

              <div className="space-y-2">
                <Label htmlFor="confirmPassword">Xác nhận mật khẩu mới</Label>
                <Input
                  id="confirmPassword"
                  type="password"
                  placeholder="••••••••"
                  value={confirmPassword}
                  onChange={(e) => setConfirmPassword(e.target.value)}
                  required
                  disabled={loading}
                />
              </div>

              <Button type="submit" className="w-full" disabled={loading}>
                {loading ? 'Đang đặt lại...' : 'Đặt lại mật khẩu'}
              </Button>

              <div className="text-center text-sm text-gray-600">
                <button
                  type="button"
                  onClick={() => setStep('request')}
                  className="text-blue-600 hover:underline"
                >
                  Quay lại yêu cầu đặt lại
                </button>
              </div>
            </form>
          </CardContent>
        </Card>
      </div>
    );
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-50 to-indigo-100 px-4">
      <Card className="w-full max-w-md">
        <CardHeader className="space-y-1">
          <CardTitle className="text-2xl font-bold text-center">Quên mật khẩu</CardTitle>
          <CardDescription className="text-center">
            Nhập email của bạn để nhận mã đặt lại mật khẩu
          </CardDescription>
        </CardHeader>
        <CardContent>
          <form onSubmit={handleRequestReset} className="space-y-4">
            {error && (
              <Alert variant="destructive">
                <AlertDescription>{error}</AlertDescription>
              </Alert>
            )}

            {success && (
              <Alert className="bg-green-50 border-green-200">
                <AlertDescription className="text-green-800">{success}</AlertDescription>
              </Alert>
            )}

            <div className="space-y-2">
              <Label htmlFor="email">Email</Label>
              <Input
                id="email"
                type="email"
                placeholder="you@example.com"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                required
                disabled={loading}
              />
            </div>

            <Button type="submit" className="w-full" disabled={loading}>
              {loading ? 'Đang gửi...' : 'Gửi mã đặt lại'}
            </Button>

            <div className="text-center text-sm text-gray-600">
              Nhớ mật khẩu của bạn?{' '}
              <Link href="/signin" className="text-blue-600 hover:underline font-medium">
                Đăng nhập
              </Link>
            </div>
          </form>
        </CardContent>
      </Card>
    </div>
  );
}


```

### 5. Trang Dashboard – `app/dashboard/page.tsx`

```tsx
'use client';

import { ProtectedRoute } from "@/components/ProtectedRoute";
import { useAuth } from "@/context/AuthContext";
import { LogOut, User, Mail, Shield, KeyRound, ArrowRight } from "lucide-react";
import Link from "next/link";

export default function DashboardPage() {
  const { user, signOut } = useAuth();

  const handleSignOut = async () => {
    await signOut();
  };

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
            <button onClick={handleSignOut} className="inline-flex items-center gap-2 px-4 py-2 rounded-lg text-sm font-medium bg-destructive/10 text-destructive hover:bg-destructive/20 transition-colors">
              <LogOut className="w-4 h-4" />
              Đăng xuất
            </button>
          </div>
        </header>

        {/* Nội dung chính */}
        <main className="max-w-6xl mx-auto px-6 py-12">
          {/* Chào mừng + Thống kê */}
          <div className="mb-12">
            <div className="space-y-2 mb-8">
              <h2 className="text-4xl font-bold text-foreground">Chào mừng trở lại</h2>
              <p className="text-lg text-muted-foreground">{user?.name || user?.email}</p>
            </div>
            <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
              {/* Trạng thái tài khoản */}
              <div className="group relative overflow-hidden rounded-2xl bg-card border border-border/50 p-6 hover:shadow-lg transition-shadow duration-300">
                <div className="absolute inset-0 bg-gradient-to-br from-primary/5 to-transparent opacity-0 group-hover:opacity-100 transition-opacity" />
                <div className="relative space-y-4">
                  <div className="w-12 h-12 rounded-xl bg-primary/10 flex items-center justify-center">
                    <Shield className="w-6 h-6 text-primary" />
                  </div>
                  <div>
                    <h3 className="text-sm font-semibold text-muted-foreground uppercase tracking-wide">Trạng thái tài khoản</h3>
                    <p className="text-2xl font-bold text-foreground mt-2">Hoạt động</p>
                  </div>
                </div>
              </div>
              {/* Vai trò */}
              <div className="group relative overflow-hidden rounded-2xl bg-card border border-border/50 p-6 hover:shadow-lg transition-shadow duration-300">
                <div className="absolute inset-0 bg-gradient-to-br from-accent/5 to-transparent opacity-0 group-hover:opacity-100 transition-opacity" />
                <div className="relative space-y-4">
                  <div className="w-12 h-12 rounded-xl bg-accent/10 flex items-center justify-center">
                    <User className="w-6 h-6 text-accent" />
                  </div>
                  <div>
                    <h3 className="text-sm font-semibold text-muted-foreground uppercase tracking-wide">Vai trò người dùng</h3>
                    <p className="text-2xl font-bold text-foreground mt-2 capitalize">{user?.role || "Thành viên"}</p>
                  </div>
                </div>
              </div>
              {/* Trạng thái Email */}
              <div className="group relative overflow-hidden rounded-2xl bg-card border border-border/50 p-6 hover:shadow-lg transition-shadow duration-300">
                <div className="absolute inset-0 bg-gradient-to-br from-chart-1/5 to-transparent opacity-0 group-hover:opacity-100 transition-opacity" />
                <div className="relative space-y-4">
                  <div className="w-12 h-12 rounded-xl bg-chart-1/10 flex items-center justify-center">
                    <Mail className="w-6 h-6 text-chart-1" />
                  </div>
                  <div>
                    <h3 className="text-sm font-semibold text-muted-foreground uppercase tracking-wide">Trạng thái Email</h3>
                    <p className="text-2xl font-bold text-foreground mt-2">Đã xác minh</p>
                  </div>
                </div>
              </div>
            </div>
          </div>

          {/* Thông tin người dùng + Hành động */}
          <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
            {/* Card chính */}
            <div className="lg:col-span-2 rounded-2xl bg-card border border-border/50 overflow-hidden hover:shadow-lg transition-shadow duration-300">
              <div className="h-24 bg-gradient-to-r from-primary to-accent/50" />
              <div className="p-8 -mt-12 relative">
                <div className="mb-8">
                  <div className="w-24 h-24 rounded-2xl bg-card border-4 border-background shadow-lg flex items-center justify-center mb-6">
                    <div className="w-full h-full bg-gradient-to-br from-primary/20 to-accent/20 rounded-xl flex items-center justify-center">
                      <User className="w-12 h-12 text-primary" />
                    </div>
                  </div>
                  <h3 className="text-2xl font-bold text-foreground">{user?.name || user?.email}</h3>
                  <p className="text-muted-foreground mt-1">Người dùng đã xác thực</p>
                </div>
                <div className="space-y-4 pt-8 border-t border-border/50">
                  <div className="flex items-start gap-4">
                    <Mail className="w-5 h-5 text-primary mt-1 flex-shrink-0" />
                    <div className="flex-1 min-w-0">
                      <p className="text-sm text-muted-foreground">Địa chỉ Email</p>
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
                      <p className="text-sm text-muted-foreground">Vai trò tài khoản</p>
                      <p className="text-foreground font-medium capitalize">{user?.role || "Thành viên"}</p>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            {/* Hành động */}
            <div className="space-y-4">
              <Link href="/forgot-password" className="flex items-center justify-between gap-3 p-4 rounded-xl bg-card border border-border/50 hover:bg-secondary/50 hover:border-border transition-all duration-200 group">
                <div className="flex items-center gap-3">
                  <KeyRound className="w-5 h-5 text-primary" />
                  <span className="font-medium text-foreground">Đổi mật khẩu</span>
                </div>
                <ArrowRight className="w-4 h-4 text-muted-foreground group-hover:translate-x-1 transition-transform" />
              </Link>
              <button onClick={handleSignOut} className="w-full flex items-center justify-between gap-3 p-4 rounded-xl bg-destructive/10 border border-destructive/20 hover:bg-destructive/20 transition-all duration-200 group">
                <div className="flex items-center gap-3">
                  <LogOut className="w-5 h-5 text-destructive" />
                  <span className="font-medium text-destructive">Đăng xuất</span>
                </div>
                <ArrowRight className="w-4 h-4 text-destructive/50 group-hover:translate-x-1 transition-transform" />
              </button>
            </div>
          </div>
        </main>
      </div>
    </ProtectedRoute>
  );
}
```

Bây giờ bạn đã có một giao diện xác thực tuyệt đẹp, hoạt động hoàn hảo — chính xác như bạn đã thiết kế.

---

**Điều hướng:**
- **Trước:** [5.5.1 Tạo Authentication Functions](../5.5.1-auth-functions/)
- **Bước tiếp theo:** [5.5.3 Triển khai Protected Routes](../5.5.3-protected-routes/) → Thêm bảo vệ route và middleware xác thực
