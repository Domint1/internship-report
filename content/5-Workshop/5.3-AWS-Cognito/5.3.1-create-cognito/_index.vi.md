---
title : "Tạo Cognito User Pool"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

### Tạo Cognito User Pool

Trong phần này, bạn sẽ tạo một Cognito User Pool bằng **hai phương pháp**:

1. **AWS Management Console (GUI)**
2. **AWS CLI (Command Line Interface)**

Chọn một trong hai phương pháp tùy theo sở thích hoặc môi trường của bạn.

---

## 1. Tạo User Pool qua AWS Console

Đây là phương pháp tiêu chuẩn sử dụng giao diện web AWS.

---

### Bước 1 — Mở Cognito Console

Truy cập:

[https://console.aws.amazon.com/cognito](https://console.aws.amazon.com/cognito)

---

### Bước 2 — Tạo User Pool mới

![hosted zone](/images/5-Workshop/5.3/image1.png)

- Nhấp **Create user pool**
- Trong **Define your application**, chọn  
  **Traditional web application**

---

### Bước 3 — Cấu hình chi tiết User Pool

- Tên ứng dụng người dùng:  
  ```text
  My web app - Cognito

---

### Bước 4 — Tùy chọn đăng nhập
![hosted zone](/images/5-Workshop/5.3/image2.png)

Chọn: Email
(Người dùng sẽ đăng nhập bằng địa chỉ email)

---
### Bước 5 — Thuộc tính bắt buộc

Thuộc tính bắt buộc:

email

---

### Bước 6 — Tạo User Pool

Nhấp Create user directory để hoàn tất

---

## 2. Tạo User Pool qua AWS CLI

Tùy chọn này hữu ích nếu:

 - Bạn muốn tự động hóa,

 - Bạn đang xây dựng pipeline IaC,

 - Hoặc bạn thích thiết lập qua dòng lệnh.

### Bước 1 Mở powershell trên AWS web và Tạo User Pool

```bash
aws cognito-idp create-user-pool \
  --pool-name "my-userpool" \
  --auto-verified-attributes email \
  --username-attributes email
```
Lệnh này sẽ:

 - Tạo một user pool mới có tên my-userpool

 - Kích hoạt email làm username

 - Kích hoạt tự động xác minh email

### Bước 2 — Tạo App Client

Chạy:
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

Thay thế:

<YOUR_USERPOOL_ID>


với giá trị được trả về từ Bước 1.

### Bước 3 — Ví dụ đầu ra
```powershell
Việc tạo thành công trả về JSON:

{
  "UserPool": {
    "Id": "ap-southeast-1_AbCdEf123",
    "Name": "my-userpool"
  }
}

```
Và:
```powershell
{
  "UserPoolClient": {
    "ClientId": "4ab5exampleid123",
    "ClientSecret": "xyzexamplesecret"
  }
}
```
 Hoàn thành

Bây giờ bạn đã có một Cognito User Pool được cấu hình đầy đủ được tạo qua:

- AWS Console, hoặc

- AWS CLI

---

**Điều hướng:**
- **Trước:** [5.3 Thiết lập AWS Cognito](../) 
- **Bước tiếp theo:** [5.3.2 Cấu hình App Client](../5.3.2-configure-appclient/) → Cấu hình cài đặt application client
