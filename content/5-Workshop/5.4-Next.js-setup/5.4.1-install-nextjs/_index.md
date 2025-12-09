---
title : "Install Next.js Project"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

### Prepare the Environment

In this step, you will create a new Next.js project and install required UI libraries that will be used later in the authentication interface.

---

### 1. Create a New Next.js Project

If you are starting from scratch, run the following command:

```bash
npx create-next-app@latest my-cognito-app
```
When prompted:
Ok to proceed? (y)

Choose y, then navigate into the project directory:
```bash
cd my-cognito-app
```

### 2. Install shadcn/ui (UI Component Library)

You will use shadcn/ui to build the authentication UI later.

Initialize shadcn:
```bash
npx shadcn@latest init
```

This will set up the component generator for your project.

Your environment is now ready for the next steps, where you will install Amplify SDK and configure the AWS Cognito integration.

---

**Navigation:**
- **Previous:** [5.4 Next.js Project Setup](../) 
- **Next Step:** [5.4.2 Install Amplify SDK](../5.4.2-install-amplify/) → Install and configure Amplify SDK