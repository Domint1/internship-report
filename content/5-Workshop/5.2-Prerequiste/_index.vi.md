---
title : "Các bước chuẩn bị"
date :  "`r Sys.Date()`" 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

### Các bước chuẩn bị

Trước khi bắt đầu workshop, hãy đảm bảo bạn đã chuẩn bị sẵn các yêu cầu sau.

---

## 1. Tài khoản AWS & Quyền truy cập

Bạn cần một tài khoản AWS để tạo và quản lý Cognito User Pools và làm việc với Amplify.

**Yêu cầu:**

- Một **Tài khoản AWS**
- Một **IAM user** (không phải root) với các quyền sau:
  - AmazonCognitoFullAccess  
  - IAMUserChangePassword  
  - AWSCloudFormationFullAccess  
  - AWSLambda_FullAccess (tùy chọn, cho triggers)  
  - AmazonS3FullAccess (cho hosting hoặc storage tùy chọn)

**Các bước:**
1. Tạo tài khoản tại AWS console.
2. Tạo IAM user với **Programmatic access + Management Console**.
3. Cấu hình AWS region của bạn (khuyến nghị: `us-east-1` hoặc `ap-southeast-1`).

---

## 2. Môi trường phát triển cục bộ

Máy tính của bạn cần được thiết lập để xây dựng và chạy ứng dụng Next.js + Amplify.

### **Phần mềm cần thiết**

| Công cụ | Phiên bản | Mục đích |
|---------|-----------|----------|
| **Node.js** | 18.x hoặc mới hơn | Cần thiết cho Next.js & Amplify |
| **npm** | 9.x hoặc mới hơn | Quản lý package |
| **Git** | Mới nhất | Kiểm soát phiên bản |
| **Code Editor** | VS Code (khuyến nghị) | Phát triển |
| **AWS CLI (tùy chọn)** | Mới nhất | Hữu ích cho debugging và thiết lập credentials |

### **Kiểm tra cài đặt**
```bash
node --version   # Nên là v18.x hoặc cao hơn
npm --version    # Nên là v9.x hoặc cao hơn
git --version    # Nên hiển thị phiên bản hợp lệ
```


---

**Điều hướng:**
- **Trước:** [5.1 Tổng quan Workshop](../5.1-Workshop-overview/) 
- **Bước tiếp theo:** [5.3 Thiết lập AWS Cognito](../5.3-AWS-Cognito/) → Tạo và cấu hình Cognito User Pool