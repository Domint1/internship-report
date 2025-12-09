---
title : "Configure App Client"
date :  "`r Sys.Date()`" 
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

#### Configure Application client 

1. Open Cloud Shell

```bash
aws cognito-idp create-user-pool-client \
    --user-pool-id us-east-1_xxxxxxx \
    --client-name WebApp-NoSecret \
    --no-generate-secret \
    --callback-urls '["http://localhost:3000/"]' \
    --logout-urls '["http://localhost:3000/"]' \
    --query 'UserPoolClient.ClientId' --output text
```
Example output: h1a234example123
you give userpood id to line, then enter 
you will create AppClient that do no generate secret

---

**Navigation:**
- **Previous:** [5.3.1 Create Cognito User Pool](../5.3.1-create-cognito/) 
- **Next Step:** [5.4 Next.js Project Setup](../../5.4-Next.js-setup/) → Set up Next.js and Amplify SDK



