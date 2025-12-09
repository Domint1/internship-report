---
title : "Install Amplify SDK and Configure Amplify"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

### Install Amplify SDK

After preparing your Next.js project, install the necessary Amplify packages:

```bash
npm install aws-amplify @aws-amplify/ui-react
```
### 2. Create the Amplify Configuration File

Create a new file:

src/lib/amplify-config.ts


And add the following code:

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


// Configure Amplify with SSR support
Amplify.configure(amplifyConfig, {
    ssr: true, // Important for Next.js
});
```
### 3. Add Environment Variables

Create a .env.local file and paste your Cognito IDs:

```bash
NEXT_PUBLIC_COGNITO_USER_POOL_ID=us-east-1_xxxxxxxxx
NEXT_PUBLIC_COGNITO_APP_CLIENT_ID=xxxxxxxxxxxxxxxxxxxx
``` 
---

This ensures Amplify is available across your entire application.



Amplify is now configured and ready to be used for:

Sign up

Sign in

Sign out

Session management

Protected routes

Next, you will build the UI for authentication.


---

**Navigation:**
- **Previous:** [5.4.1 Install Next.js](../5.4.1-install-nextjs/) 
- **Next Step:** [5.5 Cognito Functions](../../5.5-Cognito-function/) → Build auth functions, UI, and protected routes
