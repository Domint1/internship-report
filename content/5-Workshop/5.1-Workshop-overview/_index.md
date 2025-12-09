---
title: "Workshop Overview"
date: "2025-12-05"
weight: 1
chapter: false
pre: "<b>5.1. </b>"
---

## 5.1 Workshop Overview  
**What are Amazon Cognito and AWS Amplify? • Why use them together?**

### What is Amazon Cognito?

**Amazon Cognito** is AWS’s fully managed identity service that provides:

- User sign-up & sign-in (email/password, social logins, SAML/OIDC)
- Built-in email/phone verification
- Multi-factor authentication (MFA)
- User directory with custom attributes and groups
- Secure JWT tokens (ID token, access token, refresh token)
- Scales automatically to millions of users
- Free tier: 50,000 monthly active users

In this workshop, we use **Cognito User Pools** as our secure backend identity provider.

### What is AWS Amplify (Gen 2 / v6+)?

**AWS Amplify** is the official AWS toolkit for frontend and mobile developers.

The **new Amplify Gen 2 library (v6+)** released in 2024–2025 is completely rewritten and is now:

- Tree-shakable (only import what you use)
- Fully SSR-compatible (`ssr: true`)
- Designed specifically for Next.js App Router & React Server Components
- No more `@aws-amplify/ui-react` or Hosted UI required
- Uses the new modular `@aws-amplify/auth`, `@aws-amplify/datastore`, etc.

We will use only the **Auth category** of Amplify Gen 2 — the cleanest, most secure, and most modern way to connect a Next.js app to Cognito.

### Why Use Cognito + Amplify Gen 2 Together in 2025?

| Benefit                      | Explanation                                                                                 |
|------------------------------|---------------------------------------------------------------------------------------------|
| **Rapid & Secure Development** | Amplify abstracts all token handling, refresh logic, and Cognito API calls                |
| **SSR & App Router Ready**     | Works perfectly with `ssr: true` → no hydration errors, no `window is not defined`        |
| **No Client Secret**           | Public SPA client (no secret needed) → safe for Next.js & Vercel                           |
| **Production-Grade Security**  | Tokens never touch localStorage on server, automatic refresh, secure by default           |
| **Role-Based Access**          | Uses Cognito Groups → `cognito:groups` in ID token → easy admin/user detection             |
| **Deploy Anywhere**            | Works flawlessly on Vercel, Netlify, AWS Amplify Hosting, CloudFront + S3                  |
| **Future-Proof**               | This is the official AWS-recommended pattern from 2024 onward                               |

### What You Will Build (Exact Outcome)

A complete Next.js 14 (App Router) application with:

- Sign-up → automatic email verification code
- Sign-in → protected dashboard
- Global `AuthContext` + `useAuth()` hook
- `ProtectedRoute` component
- Clean, modern UI using shadcn/ui
- Zero security anti-patterns

### Ready?

You are now exactly on the same path that AWS solutions architects, startups, and enterprises use daily in 2025.

---

**Next Step:** [5.2 Prerequisites](../5.2-Prerequiste/) → Set up your AWS account, Node.js, and development environment

