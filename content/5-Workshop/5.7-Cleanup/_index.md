---
title: "Clean Up Resources"
date: "2025-12-05"
weight: 7
chapter: false
pre: "<b>5.7 </b>"
---

## 5.7 Clean Up Resources  
**Delete everything to avoid any charges (even tiny ones)**

You’ve successfully completed the workshop — congratulations!  
Now let’s make sure your AWS account stays **$0.00**.

## Resources You Created (and Must Delete)

| Resource                  | Location                          | Cost if Left Running                     | Must Delete? |
|---------------------------|-----------------------------------|------------------------------------------|--------------|
| Cognito User Pool         | AWS Console → Amazon Cognito      | Free tier: 50,000 MAU → still $0         | Recommended  |
| App Client (inside pool)  | Inside the User Pool              | No extra cost                            | Auto-deleted |
| Cognito Groups (admin/user) | Inside the User Pool            | No cost                                  | Auto-deleted |
| Test users (you signed up) | Inside the User Pool → Users      | No cost                                  | Optional     |

**Important:** Amazon Cognito User Pools are *free for up to 50,000 monthly active users*.  
You will *never* be charged for this workshop — but it’s still best practice to clean up.

## Step-by-Step Cleanup (Takes 60 Seconds)

1.  Go to the AWS Console → **Amazon Cognito**  
   → ⁦https://console.aws.amazon.com/cognito/home⁩

2.  Select your region (the one you used, e.g., us-east-1)

3.  You’ll see your User Pool (probably named something like my-cognito-app-pool or the default name)

4.  Click on your User Pool → **Delete user pool** (top-right button)

5.  Type delete in the confirmation box
![hosted zone](/images/5-Workshop/5.7/image3.png)

6.  Click **Delete user pool**


**Done!** Everything is permanently deleted.

## Optional: Also Delete Test Users (if you want a completely clean slate)

•  Inside the User Pool → *Users* tab → select any test accounts → *Delete*

You’re All Set!

Your AWS account is now clean and back to *$0.00*.

## Final Summary – What You’ve Accomplished

You have built a **2025-standard, production-ready authentication system** using:

•  Amazon Cognito (fully managed identity)
•  AWS Amplify Gen 2 (v6+) – the correct, modern, SSR-safe way
•  Next.js 14 App Router + React Server Components
•  Role-based access control via Cognito Groups
•  Protected routes & global auth context
•  Beautiful, accessible UI with shadcn/ui + Tailwind

This is **exactly** how professional teams at startups, enterprises, and AWS Partners build authentication today.

**You now have a complete, deployable, real-world project** you can:
•  Add to your portfolio
•  Use as a starter template
•  Deploy instantly to Vercel/Netlify
•  Show in job interviews

**You absolutely crushed this workshop.**

Now go deploy it, share it, and be proud — you've earned it!

---

**Navigation:**
- **Previous:** [5.6 Full Testing & Verification](../5.6-Testing/) 
- **Workshop Complete!** 🎉 Return to [Workshop Overview](../5.1-Workshop-overview/) or [Main Index](../../)

**End of Workshop**