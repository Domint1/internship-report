---
title : "Create a Cognito User Pool"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

### Create a Cognito User Pool

In this section, you will create a Cognito User Pool using **two methods**:

1. **AWS Management Console (GUI)**
2. **AWS CLI (Command Line Interface)**

Choose either method depending on your preference or environment.

---

## 1. Create User Pool via AWS Console

This is the standard method using the AWS web interface.

---

### Step 1 — Open Cognito Console

Go to:

[https://console.aws.amazon.com/cognito](https://console.aws.amazon.com/cognito)

---

### Step 2 — Create a New User Pool

![hosted zone](/images/5-Workshop/5.3/image1.png)

- Click **Create user pool**
- Under **Define your application**, choose  
  **Traditional web application**

---

### Step 3 — Configure User Pool Details

- User Application name:  
  ```text
  My web app - Cognito

---

### Step 4 — Sign-in Options
![hosted zone](/images/5-Workshop/5.3/image2.png)

Select: Email
(Users will log in using email addresses)

---
### Step 5 — Required Attributes

Required attributes:

email

---

### Step 6 — Create User Pool

Click Create user directory to fin

---

## 2.  Create User Pool via AWS CLI

This option is useful if:

 - You want automation,

 - You are building IaC pipelines,

 - Or you prefer command-line setup.

### Step 1 Open powershell on AWS web and Create User Pool

```bash
aws cognito-idp create-user-pool \
  --pool-name "my-userpool" \
  --auto-verified-attributes email \
  --username-attributes email
```
This command will:

 - Create a new user pool named my-userpool

 - Enable email as the username

 - Enable email auto-verification

### Step 2 — Create an App Client

Run:
```bash
aws cognito-idp create-user-pool-client \
  --user-pool-id <YOUR_USERPOOL_ID> \
  --client-name "my-app-client" \
  --generate-secret \
  --no-prevent-user-existence-errors \
  --allowed-o-auth-flows-user-pool-client \
  --allowed-o-auth-flows code \
  --allowed-o-auth-scopes "email" "openid" \
  --callback-urls "http://localhost:3000" \
  --logout-urls "http://localhost:3000"
```

Replace:

<YOUR_USERPOOL_ID>


with the value returned from Step 1.

### Step 3 — Output Example
```powershell
A successful creation returns JSON:

{
  "UserPool": {
    "Id": "ap-southeast-1_AbCdEf123",
    "Name": "my-userpool"
  }
}

```
And:
```powershell
{
  "UserPoolClient": {
    "ClientId": "4ab5exampleid123",
    "ClientSecret": "xyzexamplesecret"
  }
}
```
 Completed

You now have a fully configured Cognito User Pool created via:

- AWS Console, or

- AWS CLI

---

**Navigation:**
- **Previous:** [5.3 AWS Cognito Setup](../) 
- **Next Step:** [5.3.2 Configure App Client](../5.3.2-configure-appclient/) → Configure application client settings