---
title: "5.7 Dọn dẹp tài nguyên"
date: "2025-12-05"
weight: 7
chapter: false
pre: "<b>5.7 </b>"
---

## 5.7 Dọn dẹp tài nguyên  
**Xóa mọi thứ để tránh bất kỳ phí nào (ngay cả những khoản nhỏ)**

Bạn đã hoàn thành thành công workshop — chúc mừng!  
Bây giờ hãy đảm bảo tài khoản AWS của bạn vẫn là **$0.00**.

## Tài nguyên bạn đã tạo (và phải xóa)

| Tài nguyên                | Vị trí                            | Chi phí nếu để chạy                      | Phải xóa? |
|---------------------------|-----------------------------------|------------------------------------------|-----------|
| Cognito User Pool         | AWS Console → Amazon Cognito      | Free tier: 50,000 MAU → vẫn $0          | Khuyến nghị |
| App Client (trong pool)   | Bên trong User Pool               | Không tốn phí thêm                       | Tự động xóa |
| Cognito Groups (admin/user) | Bên trong User Pool             | Không tốn phí                            | Tự động xóa |
| Test users (bạn đã đăng ký) | Bên trong User Pool → Users      | Không tốn phí                            | Tùy chọn |

**Quan trọng:** Amazon Cognito User Pools *miễn phí cho tối đa 50,000 người dùng hoạt động hàng tháng*.  
Bạn sẽ *không bao giờ* bị tính phí cho workshop này — nhưng vẫn nên dọn dẹp theo thực hành tốt nhất.

## Dọn dẹp từng bước (Mất 60 giây)

1.  Đi đến AWS Console → **Amazon Cognito**  
   → ⁦https://console.aws.amazon.com/cognito/home⁩

2.  Chọn region của bạn (cái bạn đã sử dụng, ví dụ: us-east-1)

3.  Bạn sẽ thấy User Pool của mình (có thể có tên như my-cognito-app-pool hoặc tên mặc định)

4.  Click vào User Pool của bạn → **Delete user pool** (nút ở góc trên-phải)

5.  Nhập delete vào hộp xác nhận
![hosted zone](/images/5-Workshop/5.7/image3.png)

6.  Click **Delete user pool**


**Xong!** Mọi thứ đã được xóa vĩnh viễn.

## Tùy chọn: Cũng xóa Test Users (nếu bạn muốn hoàn toàn sạch sẽ)

•  Bên trong User Pool → tab *Users* → chọn bất kỳ tài khoản test nào → *Delete*

Bạn đã hoàn thành!

Tài khoản AWS của bạn bây giờ đã sạch sẽ và trở lại *$0.00*.

## Tóm tắt cuối cùng – Những gì bạn đã hoàn thành

Bạn đã xây dựng một **hệ thống xác thực chuẩn 2025, sẵn sàng cho production** sử dụng:

•  Amazon Cognito (quản lý danh tính đầy đủ)
•  AWS Amplify Gen 2 (v6+) – cách hiện đại, chính xác, an toàn với SSR
•  Next.js 14 App Router + React Server Components
•  Kiểm soát truy cập dựa trên vai trò qua Cognito Groups
•  Protected routes & global auth context
•  UI đẹp, dễ truy cập với shadcn/ui + Tailwind

Đây là **chính xác** cách các đội chuyên nghiệp tại startup, doanh nghiệp và AWS Partners xây dựng xác thực ngày nay.

**Bây giờ bạn có một dự án hoàn chỉnh, có thể deploy, thực tế** mà bạn có thể:
•  Thêm vào portfolio của mình
•  Sử dụng như template khởi đầu
•  Deploy ngay lập tức lên Vercel/Netlify
•  Trình bày trong phỏng vấn việc làm

**Bạn đã hoàn thành xuất sắc workshop này.**

Bây giờ hãy deploy nó, chia sẻ nó và tự hào — bạn đã xứng đáng!

---

**Điều hướng:**
- **Trước:** [5.6 Kiểm tra & Xác minh đầy đủ](../5.6-Testing/) 
- **Workshop hoàn thành!** 🎉 Quay lại [Tổng quan Workshop](../5.1-Workshop-overview/) hoặc [Main Index](../../)

**Kết thúc Workshop**