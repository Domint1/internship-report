---
title: "Create Authentication Functions"
date: "2025-12-05"
weight: 1
chapter: false
pre: "<b>5.5.1 </b>"
---

## 5.5.1 Create Authentication Functions  
**The complete, production-ready Cognito logic — exactly as used in your workshop**

You are now going to create the **heart of your authentication system** — a single, well-organized file containing all Cognito operations with proper error handling, TypeScript types, and role detection.

### Step 1 – Fix Folder Name & Create Config (Already Done)
Make sure you have:
- `lib/amplify-config.ts` (with `ssr: true`)
- Folder name corrected: `lib/` (not `libc/`)

### Step 2 – Create the Full Authentication Module  
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

// This import triggers Amplify configuration (side effect)
import '@/lib/amplify-config';

// Optional: safety check (useful during development)
function ensureAmplifyConfigured() {
  const config = Amplify.getConfig();
  if (!config.Auth?.Cognito?.userPoolId) {
    console.error('Amplify not configured properly. Missing userPoolId.');
    throw new Error('Auth UserPool not configured');
  }
}

// ============================================
// SIGN UP
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
      message: 'Sign up successful! Check your email for verification.',
    };
  } catch (error: any) {
    console.error('Sign up error:', error);
    if (error.name === 'UsernameExistsException') {
      return { success: false, error: 'Account already exists.' };
    }
    return { success: false, error: error.message || 'Sign up failed' };
  }
}

// ============================================
// CONFIRM SIGN UP (Email Verification)
// ============================================
export async function cognitoConfirmSignUp(email: string, code: string) {
  try {
    await confirmSignUp({
      username: email.trim(),
      confirmationCode: code.trim(),
    });
    return { success: true, message: 'Email verified! You can now sign in.' };
  } catch (error: any) {
    console.error('Confirm sign up error:', error);
    return { success: false, error: error.message || 'Verification failed' };
  }
}

// ============================================
// SIGN IN
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
        message: 'Sign in successful!',
      };
    }

    return { success: false, nextStep, error: 'Additional steps required' };
  } catch (error: any) {
    console.error('Sign in error:', error);
    if (error.name === 'NotAuthorizedException') {
      return { success: false, error: 'Invalid email or password' };
    }
    if (error.name === 'UserNotConfirmedException') {
      return { success: false, error: 'Please verify your email first' };
    }
    return { success: false, error: error.message || 'Sign in failed' };
  }
}

// ============================================
// SIGN OUT
// ============================================
export async function cognitoSignOut() {
  try {
    await signOut();
    return { success: true, message: 'Signed out successfully' };
  } catch (error: any) {
    console.error('Sign out error:', error);
    return { success: false, error: error.message || 'Sign out failed' };
  }
}

// ============================================
// GET CURRENT USER + ROLE DETECTION
// ============================================
export async function getCognitoUser() {
  try {
    // This will throw if not authenticated
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
    // Expected when user is not signed in
    if (error.name === 'UserUnAuthenticatedException' || error.message?.includes('not authenticated')) {
      return null;
    }
    console.error('Get user error:', error);
    return null;
  }
}

// ============================================
// PASSWORD RESET (Bonus)
// ============================================
export async function cognitoResetPassword(email: string) {
  try {
    await resetPassword({ username: email.trim() });
    return { success: true, message: 'Reset code sent to your email' };
  } catch (error: any) {
    return { success: false, error: error.message || 'Failed to send reset code' };
  }
}

export async function cognitoConfirmResetPassword(email: string, code: string, newPassword: string) {
  try {
    await confirmResetPassword({
      username: email.trim(),
      confirmationCode: code.trim(),
      newPassword,
    });
    return { success: true, message: 'Password reset successful!' };
  } catch (error: any) {
    return { success: false, error: error.message || 'Failed to reset password' };
  }
}
```


---

**Navigation:**
- **Previous:** [5.5 Cognito Functions Overview](../)
- **Next Step:** [5.5.2 Build Authentication UI](../5.5.2-auth-ui/) → Create sign-up, sign-in, and dashboard pages
