---
title : "Cấu hình App Client"
date :  "`r Sys.Date()`" 
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

#### Cấu hình Application client 

1. Mở Cloud Shell

```bash
aws cognito-idp create-user-pool-client \
    --user-pool-id us-east-1_xxxxxxx \
    --client-name WebApp-NoSecret \
    --no-generate-secret \
    --callback-urls '["http://localhost:3000/"]' \
    --logout-urls '["http://localhost:3000/"]' \
    --query 'UserPoolClient.ClientId' --output text
```
Ví dụ đầu ra: h1a234example123
bạn cung cấp userpood id vào dòng, sau đó nhấn enter 
bạn sẽ tạo AppClient mà không tạo secret

---

**Điều hướng:**
- **Trước:** [5.3.1 Tạo Cognito User Pool](../5.3.1-create-cognito/) 
- **Bước tiếp theo:** [5.4 Thiết lập dự án Next.js](../../5.4-Next.js-setup/) → Thiết lập Next.js và Amplify SDK