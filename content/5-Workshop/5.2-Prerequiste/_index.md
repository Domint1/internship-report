---
title : "Prerequiste"
date :  "`r Sys.Date()`" 
weight : 2 
chapter : false
pre : " <b> 5.2. </b> "
---

### Prerequisites

Before starting the workshop, ensure you have the following requirements ready.

---

## 1. AWS Account & Permissions

You need an AWS account to create and manage Cognito User Pools and work with Amplify.

**Requirements:**

- An **AWS Account**
- An **IAM user** (not root) with the following permissions:
  - AmazonCognitoFullAccess  
  - IAMUserChangePassword  
  - AWSCloudFormationFullAccess  
  - AWSLambda_FullAccess (optional, for triggers)  
  - AmazonS3FullAccess (for optional hosting or storage)

**Steps:**
1. Create an account at the AWS console.
2. Create an IAM user with **Programmatic access + Management Console**.
3. Configure your AWS region (recommended: `us-east-1` or `ap-southeast-1`).

---

## 2. Local Development Environment

Your local machine should be set up for building and running a Next.js + Amplify application.

### **Required Software**

| Tool | Version | Purpose |
|------|---------|---------|
| **Node.js** | 18.x or later | Required for Next.js & Amplify |
| **npm** | 9.x or later | Package management |
| **Git** | Latest | Version control |
| **Code Editor** | VS Code (recommended) | Development |
| **AWS CLI (optional)** | Latest | Useful for debugging and credentials setup |

### **Verify installation**
```bash
node --version   # Should be v18.x or higher
npm --version    # Should be v9.x or higher
git --version    # Should show a valid version
```

---

**Navigation:**
- **Previous:** [5.1 Workshop Overview](../5.1-Workshop-overview/) 
- **Next Step:** [5.3 AWS Cognito Setup](../5.3-AWS-Cognito/) → Create and configure Cognito User Pool
