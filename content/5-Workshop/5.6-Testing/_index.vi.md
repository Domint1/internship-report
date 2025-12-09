---
title: "Kiểm tra & Xác minh đầy đủ"
date: "2025-12-05"
weight: 6
chapter: false
pre: "<b>5.6 </b>"
---

## 5.6 Kiểm tra & Xác minh đầy đủ

**Ứng dụng của bạn đã sẵn sàng — hãy chứng minh nó hoạt động hoàn hảo**

### 1. Khởi động ứng dụng

```bash
npm run dev

```

Mở → http://localhost:3000

### 2 Kiểm tra & Xác minh đầy đủ – Danh sách kiểm tra (Đánh dấu khi hoàn thành!)

| Xong | #   | Hành động                                         | Kết quả mong đợi                                                                    |
| ---- | --- | ------------------------------------------------- | ----------------------------------------------------------------------------------- |
| ☐    | 1   | Mở trang chủ (`/`)                                | Card chào mừng với nút **Đăng nhập** và **Đăng ký**                                 |
| ☐    | 2   | Click **Đăng ký** → đi đến `/signup`              | Form đăng ký gọn gàng xuất hiện                                                     |
| ☐    | 3   | Đăng ký với **email mới** + mật khẩu mạnh         | Thành công → chuyển sang màn hình **"Xác minh Email"**                              |
| ☐    | 4   | Kiểm tra email của bạn (bao gồm Spam/Promotions)  | Nhận **mã xác minh 6 chữ số** từ Amazon Cognito                                     |
| ☐    | 5   | Nhập mã                                           | Thành công → tự động chuyển hướng đến trang **Đăng nhập**                           |
| ☐    | 6   | Đăng nhập với cùng thông tin đăng nhập            | Thành công → chuyển hướng đến **Dashboard**                                         |
| ☐    | 7   | Dashboard tải                                     | Hiển thị **tên**, **email**, **role = user** của bạn và layout đẹp                  |
| ☐    | 8   | Click **Đăng xuất**                               | Đăng xuất → quay lại trang chủ                                                      |
| ☐    | 9   | Mở tab mới → đi trực tiếp đến `/dashboard`        | **Ngay lập tức chuyển hướng** đến `/signin` → **ProtectedRoute hoạt động hoàn hảo** |
| ☐    | 10  | Thử đăng ký với **cùng email một lần nữa**        | Lỗi: **"Tài khoản đã tồn tại."**                                                    |
| ☐    | 11  | Đăng nhập với **sai mật khẩu**                    | Lỗi: **"Email hoặc mật khẩu không hợp lệ"**                                         |
| ☐    | 12  | Tạo user mới → **không xác minh** → thử đăng nhập | Lỗi: **"Vui lòng xác minh email của bạn trước"**                                    |

### 3. Kiểm tra đổi mật khẩu

| Xong | #   | Hành động                                                 | Kết quả mong đợi                                                                       |
| ---- | --- | --------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| ☐    | 13  | Đăng nhập với tài khoản đã xác minh                       | Đăng nhập thành công vào Dashboard                                                     |
| ☐    | 14  | Điều hướng đến phần **Đổi mật khẩu**                      | Form với các trường **Mật khẩu hiện tại**, **Mật khẩu mới**, **Xác nhận mật khẩu mới** |
| ☐    | 15  | Submit với **mật khẩu hiện tại sai**                      | Lỗi: **"Mật khẩu hiện tại không đúng"**                                                |
| ☐    | 16  | Submit với **mật khẩu mới yếu**                           | Lỗi: **"Mật khẩu không đáp ứng yêu cầu"**                                              |
| ☐    | 17  | Submit với **xác nhận mật khẩu không khớp**               | Lỗi: **"Mật khẩu không khớp"**                                                         |
| ☐    | 18  | Submit với **mật khẩu hiện tại đúng + mật khẩu mới mạnh** | Thành công: **"Đổi mật khẩu thành công"**                                              |
| ☐    | 19  | Đăng xuất và đăng nhập với **mật khẩu cũ**                | Lỗi: **"Email hoặc mật khẩu không hợp lệ"**                                            |
| ☐    | 20  | Đăng nhập với **mật khẩu mới**                            | Thành công → Dashboard tải chính xác                                                   |

### 4. Kiểm tra quên mật khẩu

| Xong | #   | Hành động                                      | Kết quả mong đợi                                                           |
| ---- | --- | ---------------------------------------------- | -------------------------------------------------------------------------- |
| ☐    | 21  | Đến trang Đăng nhập → Click **Quên mật khẩu?** | Chuyển hướng đến trang **Quên mật khẩu**                                   |
| ☐    | 22  | Nhập **địa chỉ email đã đăng ký**              | Thành công: **"Mã đặt lại đã được gửi đến email của bạn"**                 |
| ☐    | 23  | Kiểm tra email để có **mã đặt lại mật khẩu**   | Nhận **mã đặt lại 6 chữ số** từ Amazon Cognito                             |
| ☐    | 24  | Nhập mã đặt lại + **mật khẩu mạnh mới**        | Thành công: **"Đặt lại mật khẩu thành công"** → chuyển hướng đến Đăng nhập |
| ☐    | 25  | Đăng nhập với **mật khẩu mới**                 | Thành công → Dashboard tải chính xác                                       |

**Tất cả 25 kiểm tra đã pass?** → Bạn vừa xây dựng một **hệ thống xác thực hoạt động 100%, chuẩn production** với quản lý mật khẩu hoàn chỉnh!

khi bạn truy cập dashboard như hình này.
![hosted zone](/images/5-Workshop/5.6/image4.png)

Bạn đã chính thức hoàn thành — hãy deploy nó và cho thế giới thấy!

---

**Điều hướng:**

- **Trước:** [5.5 Authentication Functions](../5.5-Cognito-function/)
- **Bước tiếp theo:** [5.7 Dọn dẹp tài nguyên](../5.7-Cleanup/) → Xóa tài nguyên AWS để tránh phí

**Tiếp theo → 5.7 Dọn dẹp tài nguyên** (tùy chọn nhưng được khuyến nghị)
