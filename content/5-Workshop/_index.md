---
title: "Workshop"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# AWS Cognito + Amplify SDK Workshop

## Overview

**Amazon Cognito** is a fully managed user authentication, authorization, and user management service that makes it easy to add sign-up, sign-in, and access control to web and mobile apps.

In this hands-on workshop, you will:

- Create and configure a **Cognito User Pool** from scratch
- Integrate the latest **AWS Amplify Gen 2 (v6+)** with **Next.js 14 (App Router)**
- Implement a complete authentication flow:  
  Sign-up → Email verification → Sign-in → Protected routes → Role-based access
- Follow production-ready best practices (SSR support, secure token handling, clean architecture)

**Final outcome:** A fully functional, secure Next.js application with user authentication — ready to deploy on Vercel, Netlify, or AWS Amplify Hosting.

## Workshop Contents

1. **[Workshop Overview](5.1-Workshop-overview/)**
   - What are Amazon Cognito and AWS Amplify?
   - Why use them together?

2. **[Prerequisites](5.2-Prerequiste/)**
   - AWS account, IAM user, AWS CLI
   - Node.js 18+, npm, VS Code

3. **[AWS Cognito Setup](5.3-AWS-Cognito/)**
   - Create User Pool with email sign-in
   - Configure password policy, optional MFA, App Client (no client secret)
   - Create Cognito Groups (admin / user)

4. **[Next.js Project Setup & Amplify Configuration](5.4-Next.js-setup/)**
   - Scaffold Next.js 14 with App Router + TypeScript + Tailwind CSS
   - Install `aws-amplify@latest`
   - Configure environment variables and Amplify SSR mode

5. **[Authentication Functions, UI, and Protected Routes](5.5-Cognito-function/)**
   - Complete Cognito helpers: signUp, confirmSignUp, signIn, signOut, getCurrentUser, etc.
   - Global AuthContext + custom `useAuth` hook
   - Sign-up, email verification, sign-in, and forgot password pages
   - ProtectedRoute component
   - Dashboard page displaying user info and role

6. **[Full Testing & Verification](5.6-Testing/)**
   - End-to-end test checklist
   - Common errors and troubleshooting tips

7. **[Clean Up Resources](5.7-Cleanup/)**
   - Delete User Pool and App Client to avoid unwanted charges

---

**Estimated duration:** 3–4 hours  
**Difficulty:** Intermediate (basic React/Next.js knowledge recommended)

Let’s get started! → Click on 5.1 **[Workshop Overview](5.1-Workshop-overview/)** to begin.